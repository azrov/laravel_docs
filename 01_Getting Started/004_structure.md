# Laravel 12 Dokümantasyonu: Dizin Yapısı

## Giriş

Varsayılan Laravel uygulama yapısı, hem büyük hem de küçük uygulamalar için harika bir başlangıç noktası sağlamayı amaçlar. Ancak uygulamanızı dilediğiniz gibi düzenlemekte özgürsünüz. Laravel, herhangi bir sınıfın nerede bulunacağı konusunda neredeyse hiçbir kısıtlama getirmez; yeter ki Composer o sınıfı otomatik yükleyebilsin (autoload).

## Kök Dizin (The Root Directory)

### App Dizini
`app` dizini, uygulamanızın temel kodunu içerir. Bu dizini birazdan daha ayrıntılı inceleyeceğiz; ancak uygulamanızdaki sınıfların neredeyse tamamı bu dizinde olacaktır.

### Bootstrap Dizini
`bootstrap` dizini, framework'ü başlatan (bootstraps) `app.php` dosyasını içerir. Bu dizin ayrıca, rota ve servis önbellek dosyaları gibi performans optimizasyonu için framework tarafından oluşturulan dosyaları barındıran bir `cache` dizini de içerir.

### Config Dizini
`config` dizini, adından da anlaşılacağı gibi, uygulamanızın tüm yapılandırma dosyalarını içerir. Tüm bu dosyaları okumak ve size sunulan tüm seçeneklere aşina olmak harika bir fikirdir.

### Database Dizini
`database` dizini, veritabanı migration'larınızı (göçlerinizi), model fabrikalarınızı (factories) ve tohumlarınızı (seeds) içerir. İsterseniz, bir SQLite veritabanını tutmak için de bu dizini kullanabilirsiniz.

### Public Dizini
`public` dizini, uygulamanıza giren tüm isteklerin giriş noktası (entry point) olan ve otomatik yüklemeyi (autoloading) yapılandıran `index.php` dosyasını içerir. Bu dizin ayrıca resimler, JavaScript ve CSS gibi varlıklarınızı (assets) da barındırır.

### Resources Dizini
`resources` dizini, görünümlerinizin (views) yanı sıra CSS veya JavaScript gibi ham, derlenmemiş varlıklarınızı içerir.

### Routes Dizini
`routes` dizini, uygulamanız için tüm rota tanımlarını içerir. Varsayılan olarak, Laravel ile birlikte iki rota dosyası gelir: `web.php` ve `console.php`.

*   `web.php` dosyası, oturum durumu (session state), CSRF koruması ve çerez şifrelemesi sağlayan `web` ara yazılım (middleware) grubuna yerleştirilen rotaları içerir. Uygulamanız durumsuz (stateless), RESTful bir API sunmuyorsa, tüm rotalarınız büyük olasılıkla `web.php` dosyasında tanımlanacaktır.
*   `console.php` dosyası, tüm kapatma tabanlı (closure-based) konsol komutlarınızı tanımlayabileceğiniz yerdir. Her kapatma (closure), bir komut örneğine (command instance) bağlanarak her komutun G/Ç (IO) yöntemleriyle etkileşim kurmak için basit bir yaklaşım sağlar. Bu dosya HTTP rotalarını tanımlamasa da, uygulamanıza konsol tabanlı giriş noktaları (routes/rotalar) tanımlar. Ayrıca `console.php` dosyasında görevlerinizi zamanlayabilirsiniz (schedule).

İsteğe bağlı olarak, `install:api` ve `install:broadcasting` Artisan komutları aracılığıyla API rotaları (`api.php`) ve yayıncılık kanalları (`channels.php`) için ek rota dosyaları yükleyebilirsiniz.

*   `api.php` dosyası, durumsuz (stateless) olması amaçlanan rotaları içerir, bu nedenle uygulamaya bu rotalar aracılığıyla giren isteklerin token'lar aracılığıyla doğrulanması amaçlanır ve oturum durumuna erişimleri olmaz.
*   `channels.php` dosyası, uygulamanızın desteklediği tüm olay yayıncılığı (event broadcasting) kanallarını kaydedebileceğiniz yerdir.

### Storage Dizini
`storage` dizini, loglarınızı, derlenmiş Blade şablonlarınızı, dosya tabanlı oturumlarınızı, dosya önbelleklerinizi ve framework tarafından oluşturulan diğer dosyaları içerir. Bu dizin `app`, `framework` ve `logs` dizinlerine ayrılmıştır.

*   `app` dizini, uygulamanız tarafından oluşturulan herhangi bir dosyayı depolamak için kullanılabilir.
*   `framework` dizini, framework tarafından oluşturulan dosyaları ve önbellekleri depolamak için kullanılır.
*   `logs` dizini ise uygulamanızın log dosyalarını içerir.

`storage/app/public` dizini, profil resimleri gibi herkese açık olarak erişilebilmesi gereken, kullanıcı tarafından oluşturulan dosyaları depolamak için kullanılabilir. `public/storage` konumunda bu dizini işaret eden bir sembolik bağlantı (symbolic link) oluşturmalısınız. Bağlantıyı `php artisan storage:link` Artisan komutunu kullanarak oluşturabilirsiniz.

### Tests Dizini
`tests` dizini, otomatik testlerinizi içerir. Örnek Pest veya PHPUnit birim testleri (unit tests) ve özellik testleri (feature tests) kutudan çıktığı gibi sağlanır. Her test sınıfı `Test` soneki ile bitmelidir. Testlerinizi `/vendor/bin/pest` veya `/vendor/bin/phpunit` komutlarını kullanarak çalıştırabilirsiniz. Veya test sonuçlarınızın daha detaylı ve güzel bir temsilini isterseniz, testlerinizi `php artisan test` Artisan komutunu kullanarak çalıştırabilirsiniz.

### Vendor Dizini
`vendor` dizini, Composer bağımlılıklarınızı içerir.

## App Dizini

Uygulamanızın büyük bir kısmı `app` dizininde barındırılır. Varsayılan olarak, bu dizin `App` adı alanı (namespace) altındadır ve PSR-4 otomatik yükleme standardını kullanarak Composer tarafından otomatik yüklenir.

Varsayılan olarak `app` dizini `Http`, `Models` ve `Providers` dizinlerini içerir. Ancak zamanla, siz sınıflar oluşturmak için `make` Artisan komutlarını kullandıkça `app` dizini içinde çeşitli başka dizinler oluşturulacaktır. Örneğin, bir komut sınıfı oluşturmak için `make:command` Artisan komutunu çalıştırana kadar `app/Console` dizini var olmayacaktır.

Hem `Console` hem de `Http` dizinleri aşağıda ilgili bölümlerde daha ayrıntılı açıklanmıştır, ancak `Console` ve `Http` dizinlerini uygulamanızın temeline bir API sağlayan yapılar olarak düşünün. HTTP protokolü ve CLI (komut satırı arayüzü), uygulamanızla etkileşime geçmenin mekanizmalarıdır ancak aslında uygulama mantığını içermezler. Başka bir deyişle, bunlar uygulamanıza komut vermenin iki yoludur. `Console` dizini tüm Artisan komutlarınızı içerirken, `Http` dizini controller'larınızı, ara yazılımlarınızı (middleware) ve isteklerinizi (requests) içerir.

`app` dizinindeki sınıfların çoğu, komutlar aracılığıyla Artisan tarafından oluşturulabilir. Mevcut komutları gözden geçirmek için terminalinizde `php artisan list make` komutunu çalıştırın.

### Broadcasting Dizini
`Broadcasting` dizini, uygulamanız için tüm yayın kanalı (broadcast channel) sınıflarını içerir. Bu sınıflar `make:channel` komutu kullanılarak oluşturulur. Bu dizin varsayılan olarak mevcut değildir, ancak ilk kanalınızı oluşturduğunuzda sizin için oluşturulacaktır. Kanallar hakkında daha fazla bilgi edinmek için olay yayıncılığı (event broadcasting) dokümantasyonuna göz atın.

### Console Dizini
`Console` dizini, uygulamanız için tüm özel Artisan komutlarını içerir. Bu komutlar `make:command` komutu kullanılarak oluşturulabilir.

### Events Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `event:generate` ve `make:event` Artisan komutları tarafından sizin için oluşturulacaktır. `Events` dizini, olay (event) sınıflarını barındırır. Olaylar, belirli bir eylemin gerçekleştiğini uygulamanızın diğer bölümlerine bildirmek için kullanılabilir, bu da büyük bir esneklik ve ayrıştırma (decoupling) sağlar.

### Exceptions Dizini
`Exceptions` dizini, uygulamanız için tüm özel istisnaları (custom exceptions) içerir. Bu istisnalar `make:exception` komutu kullanılarak oluşturulabilir.

### Http Dizini
`Http` dizini, controller'larınızı, ara yazılımlarınızı (middleware) ve form isteklerinizi (form requests) içerir. Uygulamanıza gelen istekleri işlemek için gereken mantığın neredeyse tamamı bu dizine yerleştirilecektir.

### Jobs Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `make:job` Artisan komutunu çalıştırırsanız sizin için oluşturulacaktır. `Jobs` dizini, uygulamanız için sıraya konabilir (queueable) işleri (jobs) barındırır. İşler, uygulamanız tarafından sıraya konabilir veya geçerli istek yaşam döngüsü içinde senkron olarak çalıştırılabilir. Geçerli istek sırasında senkron olarak çalışan işler, bazen "komutlar" (commands) olarak adlandırılır çünkü bunlar komut deseninin (command pattern) bir uygulamasıdır.

### Listeners Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `event:generate` veya `make:listener` Artisan komutlarını çalıştırırsanız sizin için oluşturulacaktır. `Listeners` dizini, olaylarınızı (events) işleyen sınıfları içerir. Olay dinleyicileri (event listeners), bir olay örneği alır ve olayın tetiklenmesine yanıt olarak mantık yürütür. Örneğin, bir `UserRegistered` (Kullanıcı Kaydoldu) olayı, bir `SendWelcomeEmail` (Hoş Geldin E-postası Gönder) dinleyicisi tarafından işlenebilir.

### Mail Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `make:mail` Artisan komutunu çalıştırırsanız sizin için oluşturulacaktır. `Mail` dizini, uygulamanız tarafından gönderilen e-postaları temsil eden tüm sınıfları içerir. E-posta nesneleri, e-postaları oluşturmak için `Mail::send` yöntemini kullanmanın basit bir yolunu sağlar.

### Models Dizini
`Models` dizini, tüm Eloquent model sınıflarınızı içerir. Laravel ile birlikte gelen Eloquent ORM, veritabanınızla çalışmak için güzel, basit bir ActiveRecord uygulaması sağlar. Her veritabanı tablosunun, o tabloyla etkileşim kurmak için kullanılan karşılık gelen bir "Model"i vardır. Modeller, tablolarınızdaki verileri sorgulamanıza ve tabloya yeni kayıtlar eklemenize olanak tanır.

### Notifications Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `make:notification` Artisan komutunu çalıştırırsanız sizin için oluşturulacaktır. `Notifications` dizini, uygulamanız tarafından gönderilen, uygulamanız içinde meydana gelen olaylarla ilgili basit bildirimler gibi tüm "işlemsel" (transactional) bildirimleri içerir. Laravel'in bildirim özelliği, bildirimleri e-posta, Slack, SMS veya bir veritabanında depolama gibi çeşitli sürücüler üzerinden göndermeyi soyutlar.

### Policies Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `make:policy` Artisan komutunu çalıştırırsanız sizin için oluşturulacaktır. `Policies` dizini, uygulamanız için yetkilendirme politikası (authorization policy) sınıflarını içerir. Politikalar, bir kullanıcının bir kaynak üzerinde belirli bir eylemi gerçekleştirip gerçekleştiremeyeceğini belirlemek için kullanılır.

### Providers Dizini
`Providers` dizini, uygulamanız için tüm servis sağlayıcılarını (service providers) içerir. Servis sağlayıcılar, hizmetleri (services) servis kabına (service container) bağlayarak, olayları kaydederek veya gelen istekler için uygulamanızı hazırlamak üzere başka herhangi bir görevi gerçekleştirerek uygulamanızı başlatır (bootstrap).

Yeni bir Laravel uygulamasında, bu dizin zaten `AppServiceProvider`'ı içerecektir. Gerektiğinde bu dizine kendi sağlayıcılarınızı eklemekte özgürsünüz.

### Rules Dizini
Bu dizin varsayılan olarak mevcut değildir, ancak `make:rule` Artisan komutunu çalıştırırsanız sizin için oluşturulacaktır. `Rules` dizini, uygulamanız için özel doğrulama kuralı (custom validation rule) nesnelerini içerir. Kurallar, karmaşık doğrulama mantığını basit bir nesnede kapsüllemek için kullanılır. Daha fazla bilgi için doğrulama dokümantasyonuna göz atın.