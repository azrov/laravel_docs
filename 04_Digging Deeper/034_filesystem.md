# Laravel 12 Dokümantasyonu: Dosya Depolama (File Storage)

## Giriş

Laravel, Frank de Jonge tarafından geliştirilen harika Flysystem PHP paketi sayesinde güçlü bir dosya sistemi soyutlaması (filesystem abstraction) sağlar. Laravel Flysystem entegrasyonu, yerel dosya sistemleri (local filesystems), SFTP ve Amazon S3 ile çalışmak için basit sürücüler (drivers) sunar. Daha da iyisi, her sistem için API aynı kaldığından, yerel geliştirme makineniz ve üretim sunucunuz arasında bu depolama seçenekleri arasında geçiş yapmak inanılmaz derecede basittir.

## Yapılandırma (Configuration)

Laravel'in dosya sistemi yapılandırma dosyası `config/filesystems.php` konumunda bulunur. Bu dosya içinde, tüm dosya sistemi "disklerinizi" (disks) yapılandırabilirsiniz. Her disk, belirli bir depolama sürücüsünü ve depolama konumunu temsil eder. Desteklenen her sürücü için örnek yapılandırmalar yapılandırma dosyasında bulunur, böylece yapılandırmayı depolama tercihlerinizi ve kimlik bilgilerinizi (credentials) yansıtacak şekilde değiştirebilirsiniz.

`local` sürücüsü, Laravel uygulamasını çalıştıran sunucuda yerel olarak depolanan dosyalarla etkileşime girerken, `sftp` depolama sürücüsü SSH anahtar tabanlı FTP için kullanılır. `s3` sürücüsü ise Amazon'un S3 bulut depolama hizmetine yazmak için kullanılır.

İstediğiniz kadar çok disk yapılandırabilir ve hatta aynı sürücüyü kullanan birden çok diske sahip olabilirsiniz.

### Yerel (Local) Sürücü
`local` sürücüsünü kullanırken, tüm dosya işlemleri, dosya sistemleri yapılandırma dosyanızda tanımlanan kök dizine (root directory) göredir (relative). Varsayılan olarak bu değer `storage/app/private` dizini olarak ayarlanmıştır. Bu nedenle, aşağıdaki metod `storage/app/private/example.txt` dosyasına yazacaktır:
```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
```

### Genel (Public) Disk
Uygulamanızın dosya sistemleri yapılandırma dosyasında bulunan `public` diski, herkese açık olarak erişilebilmesi gereken dosyalar için tasarlanmıştır. Varsayılan olarak, `public` diski `local` sürücüsünü kullanır ve dosyalarını `storage/app/public` dizininde saklar.

`public` diskiniz `local` sürücüsünü kullanıyorsa ve bu dosyalara web'den erişilebilir hale getirmek istiyorsanız, kaynak dizin olan `storage/app/public` ile hedef dizin olan `public/storage` arasında sembolik bir bağlantı (symbolic link) oluşturmalısınız.

Sembolik bağlantıyı oluşturmak için `storage:link` Artisan komutunu kullanabilirsiniz:
```bash
php artisan storage:link
```
Bir dosya depolandıktan ve sembolik bağlantı oluşturulduktan sonra, `asset` yardımcısını kullanarak dosyalara bir URL oluşturabilirsiniz:
```blade
echo asset('storage/file.txt');
```
Dosya sistemleri yapılandırma dosyanızda ek sembolik bağlantılar yapılandırabilirsiniz. Yapılandırılan bağlantıların her biri, `storage:link` komutunu çalıştırdığınızda oluşturulacaktır:
```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```
`storage:unlink` komutu, yapılandırılmış sembolik bağlantılarınızı yok etmek için kullanılabilir:
```bash
php artisan storage:unlink
```

### Sürücü Ön Gereksinimleri (Driver Prerequisites)

#### S3 Sürücü Yapılandırması
S3 sürücüsünü kullanmadan önce, Flysystem S3 paketini Composer paket yöneticisi aracılığıyla kurmanız gerekecektir:
```bash
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
`config/filesystems.php` yapılandırma dosyanızda bir S3 disk yapılandırma dizisi bulunur. Tipik olarak, S3 bilgilerinizi ve kimlik bilgilerinizi, `config/filesystems.php` yapılandırma dosyası tarafından referans alınan aşağıdaki ortam değişkenlerini kullanarak yapılandırmalısınız:
```
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```
Kolaylık olması açısından, bu ortam değişkenleri AWS CLI tarafından kullanılan adlandırma kuralıyla (naming convention) eşleşir.

#### FTP Sürücü Yapılandırması
FTP sürücüsünü kullanmadan önce, Flysystem FTP paketini Composer paket yöneticisi aracılığıyla kurmanız gerekecektir:
```bash
composer require league/flysystem-ftp "^3.0"
```
Laravel'in Flysystem entegrasyonları FTP ile harika çalışır; ancak, framework'ün varsayılan `config/filesystems.php` yapılandırma dosyasına örnek bir yapılandırma dahil edilmemiştir. Bir FTP dosya sistemi yapılandırmanız gerekirse, aşağıdaki örnek yapılandırmayı kullanabilirsiniz:
```php
'ftp' => [
    'driver' => 'ftp',
    'host' => env('FTP_HOST'),
    'username' => env('FTP_USERNAME'),
    'password' => env('FTP_PASSWORD'),

    // İsteğe bağlı FTP Ayarları...
    // 'port' => env('FTP_PORT', 21),
    // 'root' => env('FTP_ROOT'),
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

#### SFTP Sürücü Yapılandırması
SFTP sürücüsünü kullanmadan önce, Flysystem SFTP paketini Composer paket yöneticisi aracılığıyla kurmanız gerekecektir:
```bash
composer require league/flysystem-sftp-v3 "^3.0"
```
Laravel'in Flysystem entegrasyonları SFTP ile harika çalışır; ancak, framework'ün varsayılan `config/filesystems.php` yapılandırma dosyasına örnek bir yapılandırma dahil edilmemiştir. Bir SFTP dosya sistemi yapılandırmanız gerekirse, aşağıdaki örnek yapılandırmayı kullanabilirsiniz:
```php
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),

    // Temel kimlik doğrulama için ayarlar...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),

    // Şifreleme parolasına sahip SSH anahtar tabanlı kimlik doğrulama için ayarlar...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'passphrase' => env('SFTP_PASSPHRASE'),

    // Dosya / dizin izinleri için ayarlar...
    'visibility' => 'private', // `private` = 0600, `public` = 0644
    'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

    // İsteğe bağlı SFTP Ayarları...
    // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
    // 'maxTries' => 4,
    // 'port' => env('SFTP_PORT', 22),
    // 'root' => env('SFTP_ROOT', ''),
    // 'timeout' => 30,
    // 'useAgent' => true,
],
```

### Kapsamlı (Scoped) ve Salt Okunur (Read-Only) Dosya Sistemleri
Kapsamlı diskler (scoped disks), tüm yolların belirli bir yol ön ekiyle (path prefix) otomatik olarak ön eklendiği bir dosya sistemi tanımlamanıza olanak tanır. Kapsamlı bir dosya sistemi diski oluşturmadan önce, Composer paket yöneticisi aracılığıyla ek bir Flysystem paketi kurmanız gerekecektir:
```bash
composer require league/flysystem-path-prefixing "^3.0"
```
`scoped` sürücüsünü kullanan bir disk tanımlayarak, mevcut herhangi bir dosya sistemi diskinin yol kapsamlı bir örneğini oluşturabilirsiniz. Örneğin, mevcut `s3` diskinizi belirli bir yol ön ekine göre kapsamlandıran bir disk oluşturabilir ve ardından kapsamlı diskinizi kullanan her dosya işlemi belirtilen ön eki kullanacaktır:
```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```
"Salt okunur" (read-only) diskler, yazma işlemlerine izin vermeyen dosya sistemi diskleri oluşturmanıza olanak tanır. `read-only` yapılandırma seçeneğini kullanmadan önce, Composer paket yöneticisi aracılığıyla ek bir Flysystem paketi kurmanız gerekecektir:
```bash
composer require league/flysystem-read-only "^3.0"
```
Ardından, disklerinizden birinin veya birkaçının yapılandırma dizisine `read-only` yapılandırma seçeneğini ekleyebilirsiniz:
```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

### Amazon S3 Uyumlu Dosya Sistemleri
Varsayılan olarak, uygulamanızın `filesystems.php` yapılandırma dosyası `s3` diski için bir disk yapılandırması içerir. Bu diski Amazon S3 ile etkileşim kurmak için kullanmanın yanı sıra, RustFS, DigitalOcean Spaces, Vultr Object Storage, Cloudflare R2 veya Hetzner Cloud Storage gibi S3 uyumlu herhangi bir dosya depolama hizmetiyle etkileşim kurmak için de kullanabilirsiniz.

Tipik olarak, diskin kimlik bilgilerini kullanmayı planladığınız hizmetin kimlik bilgileriyle eşleşecek şekilde güncelledikten sonra, yalnızca `endpoint` yapılandırma seçeneğinin değerini güncellemeniz gerekir. Bu seçeneğin değeri tipik olarak `AWS_ENDPOINT` ortam değişkeni aracılığıyla tanımlanır:
```php
'endpoint' => env('AWS_ENDPOINT', 'https://rustfs:9000'),
```

## Disk Örnekleri (Instances) Alma

`Storage` facade'ı, yapılandırılmış disklerinizden herhangi biriyle etkileşim kurmak için kullanılabilir. Örneğin, facade üzerindeki `put` metodunu kullanarak bir avatarı varsayılan diskte saklayabilirsiniz. `Storage` facade'ında `disk` metodunu önce çağırmadan metotları çağırırsanız, metod otomatik olarak varsayılan diske iletilecektir:
```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $content);
```
Uygulamanız birden çok diskle etkileşime giriyorsa, belirli bir diskteki dosyalarla çalışmak için `Storage` facade'ında `disk` metodunu kullanabilirsiniz:
```php
Storage::disk('s3')->put('avatars/1', $content);
```

### İsteğe Bağlı (On-Demand) Diskler
Bazen, bu yapılandırma aslında uygulamanızın `filesystems.php` yapılandırma dosyasında mevcut olmadan, çalışma zamanında belirli bir yapılandırmayı kullanarak bir disk oluşturmak isteyebilirsiniz. Bunu başarmak için `Storage` facade'ının `build` metoduna bir yapılandırma dizisi iletebilirsiniz:
```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

## Dosya Alma (Retrieving Files)

`get` metodu, bir dosyanın içeriğini almak için kullanılabilir. Metod, dosyanın ham dize içeriğini (raw string contents) döndürecektir. Unutmayın, tüm dosya yolları diskin "kök" (root) konumuna göre belirtilmelidir:
```php
$contents = Storage::get('file.jpg');
```
Aldığınız dosya JSON içeriyorsa, dosyayı almak ve içeriğini çözmek (decode) için `json` metodunu kullanabilirsiniz:
```php
$orders = Storage::json('orders.json');
```
`exists` metodu, bir dosyanın diskte mevcut olup olmadığını belirlemek için kullanılabilir:
```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```
`missing` metodu, bir dosyanın diskte olup olmadığını (mevcut olmadığını) belirlemek için kullanılabilir:
```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

### Dosya İndirme (Downloading Files)
`download` metodu, kullanıcının tarayıcısını verilen yoldaki dosyayı indirmeye zorlayan bir yanıt (response) oluşturmak için kullanılabilir. `download` metodu, ikinci argüman olarak bir dosya adı kabul eder; bu, dosyayı indiren kullanıcı tarafından görülecek dosya adını belirleyecektir. Son olarak, metoda üçüncü argüman olarak bir HTTP başlıkları (headers) dizisi iletebilirsiniz:
```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

### Dosya URL'leri
Belirli bir dosya için URL almak üzere `url` metodunu kullanabilirsiniz. `local` sürücüsünü kullanıyorsanız, bu tipik olarak verilen yola `/storage` ekler (prepend) ve dosyaya göreceli bir URL (relative URL) döndürür. `s3` sürücüsünü kullanıyorsanız, tam nitelikli uzak URL (fully qualified remote URL) döndürülecektir:
```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```
`local` sürücüsünü kullanırken, herkese açık olarak erişilebilmesi gereken tüm dosyalar `storage/app/public` dizinine yerleştirilmelidir. Ayrıca, `public/storage` konumunda `storage/app/public` dizinini işaret eden bir sembolik bağlantı (symbolic link) oluşturmalısınız.

`local` sürücüsünü kullanırken, `url`'nin dönüş değeri URL kodlu (URL encoded) değildir. Bu nedenle, geçerli URL'ler oluşturacak adlar kullanarak dosyalarınızı her zaman saklamanızı öneririz.

#### URL Ana Bilgisayarını (Host) Özelleştirme
`Storage` facade'ı kullanılarak oluşturulan URL'ler için ana bilgisayarı (host) değiştirmek isterseniz, diskin yapılandırma dizisine `url` seçeneğini ekleyebilir veya değiştirebilirsiniz:
```php
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
    'throw' => false,
],
```

### Geçici (Temporary) URL'ler
`temporaryUrl` metodunu kullanarak, `local` ve `s3` sürücüleri kullanılarak depolanan dosyalar için geçici URL'ler oluşturabilirsiniz. Bu metod, bir yol ve URL'nin ne zaman sona ermesi gerektiğini belirten bir `DateTime` örneği kabul eder:
```php
use Illuminate\Support\Facades\Storage;

$url = Storage::temporaryUrl(
    'file.jpg', now()->addMinutes(5)
);
```

#### Yerel (Local) Geçici URL'leri Etkinleştirme
Uygulamanızı geliştirmeye, `local` sürücüsü için geçici URL desteği sunulmadan önce başladıysanız, yerel geçici URL'leri etkinleştirmeniz gerekebilir. Bunu yapmak için, `config/filesystems.php` yapılandırma dosyasındaki yerel diskinizin yapılandırma dizisine `serve` seçeneğini ekleyin:
```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'serve' => true, // Yerel geçici URL'leri etkinleştirir
    'throw' => false,
],
```

#### S3 İstek Parametreleri
Ek S3 istek parametreleri belirtmeniz gerekiyorsa, istek parametreleri dizisini `temporaryUrl` metoduna üçüncü argüman olarak iletebilirsiniz:
```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->addMinutes(5),
    [
        'ResponseContentType' => 'application/octet-stream',
        'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
    ]
);
```

#### Geçici URL'leri Özelleştirme
Belirli bir depolama diski için geçici URL'lerin nasıl oluşturulacağını özelleştirmeniz gerekiyorsa, `buildTemporaryUrlsUsing` metodunu kullanabilirsiniz. Örneğin, normalde geçici URL'leri desteklemeyen bir disk aracılığıyla depolanan dosyaları indirmenize izin veren bir controller'ınız varsa bu kullanışlı olabilir. Genellikle bu metod, bir servis sağlayıcının (service provider) `boot` metodundan çağrılmalıdır:
```php
<?php

namespace App\Providers;

use DateTime;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        Storage::disk('local')->buildTemporaryUrlsUsing(
            function (string $path, DateTime $expiration, array $options) {
                return URL::temporarySignedRoute(
                    'files.download',
                    $expiration,
                    array_merge($options, ['path' => $path])
                );
            }
        );
    }
}
```

#### Geçici Yükleme (Upload) URL'leri
Geçici yükleme URL'leri oluşturma yeteneği yalnızca `s3` ve `local` sürücüleri tarafından desteklenir.

İstemci tarafı uygulamanızdan doğrudan bir dosya yüklemek için kullanılabilecek geçici bir URL oluşturmanız gerekiyorsa, `temporaryUploadUrl` metodunu kullanabilirsiniz. Bu metod, bir yol ve URL'nin ne zaman sona ermesi gerektiğini belirten bir `DateTime` örneği kabul eder. `temporaryUploadUrl` metodu, yapısı bozulabilen (destructured) ilişkisel bir dizi (associative array) döndürür; bu dizi, yükleme URL'sini ve yükleme isteğine dahil edilmesi gereken başlıkları (headers) içerir:
```php
use Illuminate\Support\Facades\Storage;

['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
    'file.jpg', now()->addMinutes(5)
);
```