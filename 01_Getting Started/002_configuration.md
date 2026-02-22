# Laravel 12 Dokümantasyonu: Yapılandırma (Configuration)

## Giriş

Laravel framework'üne ait tüm yapılandırma dosyaları `config` dizininde bulunur. Her seçenek belgelenmiştir, bu nedenle dosyaları incelemekten ve size sunulan seçeneklere aşina olmaktan çekinmeyin.

Bu yapılandırma dosyaları, veritabanı bağlantı bilgileriniz, posta sunucusu bilgileriniz, uygulama URL'niz ve şifreleme anahtarınız gibi çeşitli diğer temel yapılandırma değerlerini yapılandırmanıza olanak tanır.

#### `about` Komutu

Laravel, `about` Artisan komutu aracılığıyla uygulamanızın yapılandırması, sürücüleri ve ortamı hakkında bir genel bakış görüntüleyebilir.
```bash
php artisan about
```
Eğer sadece uygulama genel bakış çıktısının belirli bir bölümüyle ilgileniyorsanız, `--only` seçeneğini kullanarak o bölüme filtre uygulayabilirsiniz:
```bash
php artisan about --only=environment
```
Veya belirli bir yapılandırma dosyasının değerlerini detaylı bir şekilde incelemek için `config:show` Artisan komutunu kullanabilirsiniz:
```bash
php artisan config:show database
```

## Ortam Yapılandırması (Environment Configuration)

Uygulamanın çalıştığı ortama göre farklı yapılandırma değerlerine sahip olmak genellikle faydalıdır. Örneğin, yerel ortamda üretim sunucusundakinden farklı bir önbellek sürücüsü kullanmak isteyebilirsiniz.

Bunu çok kolay hale getirmek için Laravel, DotEnv PHP kütüphanesini kullanır. Yeni bir Laravel kurulumunda, uygulamanızın kök dizini, birçok yaygın ortam değişkenini tanımlayan bir `.env.example` dosyası içerecektir. Laravel kurulum işlemi sırasında, bu dosya otomatik olarak `.env`'ye kopyalanır.

Laravel'in varsayılan `.env` dosyası, uygulamanızın yerel olarak mı yoksa bir üretim web sunucusunda mı çalıştığına bağlı olarak değişebilecek bazı yaygın yapılandırma değerlerini içerir. Bu değerler daha sonra `config` dizini içindeki yapılandırma dosyaları tarafından Laravel'in `env` fonksiyonu kullanılarak okunur.

Bir ekiple geliştirme yapıyorsanız, `.env.example` dosyasını uygulamanızla birlikte eklemeye ve güncellemeye devam etmek isteyebilirsiniz. Örnek yapılandırma dosyasına yer tutucu değerler koyarak, ekibinizdeki diğer geliştiriciler uygulamanızı çalıştırmak için hangi ortam değişkenlerinin gerekli olduğunu net bir şekilde görebilir.

`.env` dosyanızdaki herhangi bir değişken, sunucu düzeyi veya sistem düzeyi ortam değişkenleri gibi harici ortam değişkenleri tarafından geçersiz kılınabilir.

#### Ortam Dosyası Güvenliği

`.env` dosyanız, uygulamanızı kullanan her geliştirici/sunucu farklı bir ortam yapılandırması gerektirebileceğinden, uygulamanızın kaynak kontrolüne (source control) eklenmemelidir. Ayrıca, bir saldırgan kaynak kontrol deponuza erişim sağlarsa, herhangi bir hassas kimlik bilgisi ifşa olacağından bu bir güvenlik riskidir.

Ancak, Laravel'in yerleşik ortam şifreleme özelliğini kullanarak ortam dosyanızı şifrelemek mümkündür. Şifrelenmiş ortam dosyaları güvenle kaynak kontrolüne eklenebilir.

#### Ek Ortam Dosyaları

Laravel, uygulamanızın ortam değişkenlerini yüklemeden önce, harici olarak bir `APP_ENV` ortam değişkeninin sağlanıp sağlanmadığını veya `--env` CLI argümanının belirtilip belirtilmediğini kontrol eder. Eğer belirtilmişse, Laravel varsa bir `.env.[APP_ENV]` dosyasını yüklemeye çalışacaktır. Eğer dosya yoksa, varsayılan `.env` dosyası yüklenecektir.

### Ortam Değişkeni Türleri

`.env` dosyalarınızdaki tüm değişkenler tipik olarak string olarak ayrıştırılır, bu nedenle `env()` fonksiyonundan daha geniş bir tür yelpazesi döndürebilmeniz için bazı ayrılmış değerler oluşturulmuştur:

| .env Değeri | env() Değeri |
| :--- | :--- |
| `true` | (bool) `true` |
| `(true)` | (bool) `true` |
| `false` | (bool) `false` |
| `(false)` | (bool) `false` |
| `empty` | (string) `''` |
| `(empty)` | (string) `''` |
| `null` | (null) `null` |
| `(null)` | (null) `null` |

Boşluk içeren bir değere sahip bir ortam değişkeni tanımlamanız gerekirse, bunu değeri çift tırnak içine alarak yapabilirsiniz:
```
APP_NAME="My Application"
```

### Ortam Yapılandırmasını Alma (Retrieving Environment Configuration)

Uygulamanız bir istek aldığında, `.env` dosyasında listelenen tüm değişkenler `$_ENV` PHP süper küreseli içine yüklenecektir. Ancak, yapılandırma dosyalarınızda bu değişkenlerden değerleri almak için `env` fonksiyonunu kullanabilirsiniz. Aslında, Laravel yapılandırma dosyalarını incelerseniz, seçeneklerin çoğunun zaten bu fonksiyonu kullandığını göreceksiniz:
```php
'debug' => (bool) env('APP_DEBUG', false),
```
`env` fonksiyonuna iletilen ikinci değer "varsayılan değer"dir. Bu değer, belirtilen anahtar için hiçbir ortam değişkeni yoksa döndürülecektir.

### Geçerli Ortamı Belirleme (Determining the Current Environment)

Geçerli uygulama ortamı, `.env` dosyanızdaki `APP_ENV` değişkeni aracılığıyla belirlenir. Bu değere `App` facade'ı üzerindeki `environment` metodu ile erişebilirsiniz:
```php
use Illuminate\Support\Facades\App;

$environment = App::environment();
```
Ayrıca, ortamın belirli bir değerle eşleşip eşleşmediğini belirlemek için `environment` metoduna argümanlar da iletebilirsiniz. Ortam, verilen değerlerden herhangi biriyle eşleşiyorsa metod `true` döndürecektir:
```php
if (App::environment('local')) {
    // Ortam local (yerel)
}

if (App::environment(['local', 'staging'])) {
    // Ortam local VEYA staging...
}
```
Geçerli uygulama ortamı algılaması, sunucu düzeyinde bir `APP_ENV` ortam değişkeni tanımlanarak geçersiz kılınabilir.

### Ortam Dosyalarını Şifreleme (Encrypting Environment Files)

Şifrelenmemiş ortam dosyaları asla kaynak kontrolünde saklanmamalıdır. Ancak Laravel, ortam dosyalarınızı şifrelemenize izin verir, böylece uygulamanızın geri kalanıyla birlikte güvenli bir şekilde kaynak kontrolüne eklenebilirler.

#### Şifreleme (Encryption)

Bir ortam dosyasını şifrelemek için `env:encrypt` komutunu kullanabilirsiniz:
```bash
php artisan env:encrypt
```
`env:encrypt` komutunu çalıştırmak, `.env` dosyanızı şifreleyecek ve şifrelenmiş içeriği bir `.env.encrypted` dosyasına yerleştirecektir. Şifre çözme anahtarı (decryption key) komutun çıktısında gösterilir ve güvenli bir parola yöneticisinde saklanmalıdır. Kendi şifreleme anahtarınızı sağlamak isterseniz, komutu çağırırken `--key` seçeneğini kullanabilirsiniz:
```bash
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```
Sağlanan anahtarın uzunluğu, kullanılan şifreleme algoritması (cipher) tarafından gereken anahtar uzunluğuyla eşleşmelidir. Varsayılan olarak Laravel, 32 karakterlik bir anahtar gerektiren `AES-256-CBC` algoritmasını kullanacaktır. Komutu çağırırken `--cipher` seçeneğini ileterek Laravel'in şifreleyicisi tarafından desteklenen herhangi bir algoritmayı kullanmakta özgürsünüz.

Uygulamanızın `.env` ve `.env.staging` gibi birden fazla ortam dosyası varsa, `--env` seçeneği aracılığıyla ortam adını belirterek şifrelenmesi gereken ortam dosyasını belirtebilirsiniz:
```bash
php artisan env:encrypt --env=staging
```

#### Okunabilir Değişken Adları (Readable Variable Names)

Ortam dosyanızı şifrelerken, değerlerini şifrelerken değişken adlarını görünür tutmak için `--readable` seçeneğini kullanabilirsiniz:
```bash
php artisan env:encrypt --readable
```
Bu, aşağıdaki formatta şifrelenmiş bir dosya üretecektir:
```
APP_NAME=eyJpdiI6...
APP_ENV=eyJpdiI6...
APP_KEY=eyJpdiI6...
APP_DEBUG=eyJpdiI6...
APP_URL=eyJpdiI6...
```
Okunabilir formatı kullanmak, hassas verileri ifşa etmeden hangi ortam değişkenlerinin mevcut olduğunu görmenizi sağlar. Ayrıca, dosyayı şifresini çözmeden hangi değişkenlerin eklendiğini, kaldırıldığını veya yeniden adlandırıldığını görebileceğiniz için pull request'leri incelemeyi çok daha kolay hale getirir.

Ortam dosyalarının şifresini çözerken, Laravel otomatik olarak hangi formatın kullanıldığını algılar, bu nedenle `env:decrypt` komutu için ek bir seçeneğe gerek yoktur.

`--readable` seçeneğini kullanırken, orijinal ortam dosyasındaki yorumlar ve boş satırlar şifrelenmiş çıktıya dahil edilmez.

#### Şifre Çözme (Decryption)

Bir ortam dosyasının şifresini çözmek için `env:decrypt` komutunu kullanabilirsiniz. Bu komut, Laravel'in `LARAVEL_ENV_ENCRYPTION_KEY` ortam değişkeninden alacağı bir şifre çözme anahtarı gerektirir:
```bash
php artisan env:decrypt
```
Veya anahtar, `--key` seçeneği aracılığıyla doğrudan komuta sağlanabilir:
```bash
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```
`env:decrypt` komutu çağrıldığında, Laravel `.env.encrypted` dosyasının içeriğinin şifresini çözecek ve çözülmüş içeriği `.env` dosyasına yerleştirecektir.

Özel bir şifreleme algoritması kullanmak için `env:decrypt` komutuna `--cipher` seçeneği sağlanabilir:
```bash
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```
Uygulamanızın `.env` ve `.env.staging` gibi birden fazla ortam dosyası varsa, `--env` seçeneği aracılığıyla ortam adını belirterek şifresi çözülmesi gereken ortam dosyasını belirtebilirsiniz:
```bash
php artisan env:decrypt --env=staging
```
Mevcut bir ortam dosyasının üzerine yazmak için `env:decrypt` komutuna `--force` seçeneğini sağlayabilirsiniz:
```bash
php artisan env:decrypt --force
```

## Yapılandırma Değerlerine Erişim (Accessing Configuration Values)

Uygulamanızın herhangi bir yerinden `Config` facade'ını veya global `config` fonksiyonunu kullanarak yapılandırma değerlerinize kolayca erişebilirsiniz. Yapılandırma değerlerine, erişmek istediğiniz dosyanın ve seçeneğin adını içeren "nokta" sözdizimi kullanılarak erişilebilir. Bir varsayılan değer de belirtilebilir ve yapılandırma seçeneği mevcut değilse bu değer döndürülür:
```php
use Illuminate\Support\Facades\Config;

$value = Config::get('app.timezone');

$value = config('app.timezone');

// Yapılandırma değeri yoksa bir varsayılan değer al...
$value = config('app.timezone', 'Asia/Seoul');
```
Yapılandırma değerlerini çalışma zamanında ayarlamak için `Config` facade'ının `set` metodunu çağırabilir veya `config` fonksiyonuna bir dizi iletebilirsiniz:
```php
Config::set('app.timezone', 'America/Chicago');

config(['app.timezone' => 'America/Chicago']);
```
Statik analize yardımcı olmak için `Config` facade'ı ayrıca tür belirtilmiş yapılandırma alma metotları sağlar. Alınan yapılandırma değeri beklenen türle eşleşmezse, bir istisna (exception) fırlatılacaktır:
```php
Config::string('config-key');
Config::integer('config-key');
Config::float('config-key');
Config::boolean('config-key');
Config::array('config-key');
Config::collection('config-key');
```

## Yapılandırma Önbelleğe Alma (Configuration Caching)

Uygulamanıza bir hız artışı sağlamak için, tüm yapılandırma dosyalarınızı `config:cache` Artisan komutunu kullanarak tek bir dosyada önbelleğe almalısınız. Bu, uygulamanız için tüm yapılandırma seçeneklerini, framework tarafından hızlıca yüklenebilecek tek bir dosyada birleştirecektir.

`php artisan config:cache` komutunu genellikle üretim dağıtım sürecinizin bir parçası olarak çalıştırmalısınız. Yerel geliştirme sırasında bu komut çalıştırılmamalıdır, çünkü uygulamanızın geliştirilmesi sırasında yapılandırma seçeneklerinin sık sık değiştirilmesi gerekecektir.

Yapılandırma önbelleğe alındıktan sonra, uygulamanızın `.env` dosyası istekler veya Artisan komutları sırasında framework tarafından yüklenmeyecektir; bu nedenle `env` fonksiyonu yalnızca harici, sistem düzeyi ortam değişkenlerini döndürecektir.

Bu nedenle, `env` fonksiyonunu yalnızca uygulamanızın yapılandırma (`config`) dosyaları içinden çağırdığınızdan emin olmalısınız. Laravel'in varsayılan yapılandırma dosyalarını inceleyerek bunun birçok örneğini görebilirsiniz. Yapılandırma değerlerine uygulamanızın herhangi bir yerinden yukarıda açıklanan `config` fonksiyonu kullanılarak erişilebilir.

Önbelleğe alınmış yapılandırmayı temizlemek için `config:clear` komutu kullanılabilir:
```bash
php artisan config:clear
```
Dağıtım süreciniz sırasında `config:cache` komutunu çalıştırırsanız, `env` fonksiyonunu yalnızca yapılandırma dosyalarınızın içinden çağırdığınızdan emin olmalısınız. Yapılandırma önbelleğe alındıktan sonra `.env` dosyası yüklenmeyecektir; bu nedenle `env` fonksiyonu yalnızca harici, sistem düzeyi ortam değişkenlerini döndürecektir.

## Yapılandırma Yayınlama (Configuration Publishing)

Laravel'in yapılandırma dosyalarının çoğu, uygulamanızın `config` dizininde zaten yayınlanmıştır; ancak `cors.php` ve `view.php` gibi bazı yapılandırma dosyaları, çoğu uygulamanın bunları asla değiştirmesine gerek kalmayacağından varsayılan olarak yayınlanmaz.

Yine de, varsayılan olarak yayınlanmayan herhangi bir yapılandırma dosyasını yayınlamak için `config:publish` Artisan komutunu kullanabilirsiniz:
```bash
php artisan config:publish

php artisan config:publish --all
```

## Hata Ayıklama Modu (Debug Mode)

`config/app.php` yapılandırma dosyanızdaki `debug` seçeneği, bir hata hakkında kullanıcıya aslında ne kadar bilgi gösterileceğini belirler. Varsayılan olarak bu seçenek, `.env` dosyanızda saklanan `APP_DEBUG` ortam değişkeninin değerine saygı gösterecek şekilde ayarlanmıştır.

Yerel geliştirme için `APP_DEBUG` ortam değişkenini `true` olarak ayarlamalısınız. Üretim ortamınızda bu değer her zaman `false` olmalıdır. Değişken üretimde `true` olarak ayarlanırsa, hassas yapılandırma değerlerini uygulamanızın son kullanıcılarına ifşa etme riskiniz vardır.

## Bakım Modu (Maintenance Mode)

Uygulamanız bakım modundayken, uygulamanıza yapılan tüm istekler için özel bir görünüm (view) görüntülenecektir. Bu, uygulamanızı güncellerken veya bakım yaparken "devre dışı bırakmayı" kolaylaştırır. Uygulamanız için varsayılan ara yazılım (middleware) yığınına bir bakım modu kontrolü dahil edilmiştir. Uygulama bakım modundaysa, 503 durum koduyla bir `Symfony\Component\HttpKernel\Exception\HttpException` örneği fırlatılacaktır.

Bakım modunu etkinleştirmek için `down` Artisan komutunu çalıştırın:
```bash
php artisan down
```
Tüm bakım modu yanıtlarıyla birlikte `Refresh` HTTP başlığının gönderilmesini isterseniz, `down` komutunu çağırırken `refresh` seçeneğini sağlayabilirsiniz. `Refresh` başlığı, tarayıcıya belirtilen saniye sayısından sonra sayfayı otomatik olarak yenilemesi talimatını verecektir:
```bash
php artisan down --refresh=15
```
Ayrıca `down` komutuna bir `retry` seçeneği de sağlayabilirsiniz; bu, `Retry-After` HTTP başlığının değeri olarak ayarlanacaktır, ancak tarayıcılar genellikle bu başlığı dikkate almaz:
```bash
php artisan down --retry=60
```

#### Bakım Modunu Atlama (Bypassing Maintenance Mode)

Bakım modunun gizli bir anahtar (secret token) kullanılarak atlanmasına izin vermek için, `secret` seçeneğini kullanarak bir bakım modu atlama anahtarı belirtebilirsiniz:
```bash
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```
Uygulamayı bakım moduna aldıktan sonra, bu anahtarla eşleşen uygulama URL'sine gidebilirsiniz ve Laravel tarayıcınıza bir bakım modu atlama çerezi (cookie) gönderecektir:
```
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```
Laravel'in sizin için gizli anahtarı oluşturmasını isterseniz, `with-secret` seçeneğini kullanabilirsiniz. Anahtar size gösterilecektir.