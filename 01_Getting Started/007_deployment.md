# Laravel 12 Dokümantasyonu: Dağıtım (Deployment)

## Giriş

Laravel uygulamanızı canlıya (production) dağıtmaya hazır olduğunuzda, uygulamanızın olabildiğince verimli çalıştığından emin olmak için yapabileceğiniz bazı önemli şeyler vardır. Bu belgede, Laravel uygulamanızın doğru bir şekilde dağıtıldığından emin olmak için harika başlangıç noktalarını ele alacağız.

## Sunucu Gereksinimleri (Server Requirements)

Laravel framework'ünün birkaç sistem gereksinimi vardır. Web sunucunuzun aşağıdaki minimum PHP sürümüne ve eklentilere (extensions) sahip olduğundan emin olmalısınız:

*   PHP >= 8.2
*   Ctype PHP Eklentisi
*   cURL PHP Eklentisi
*   DOM PHP Eklentisi
*   Fileinfo PHP Eklentisi
*   Filter PHP Eklentisi
*   Hash PHP Eklentisi
*   Mbstring PHP Eklentisi
*   OpenSSL PHP Eklentisi
*   PCRE PHP Eklentisi
*   PDO PHP Eklentisi
*   Session PHP Eklentisi
*   Tokenizer PHP Eklentisi
*   XML PHP Eklentisi

## Sunucu Yapılandırması (Server Configuration)

### Nginx Yapılandırması
Uygulamanızı Nginx çalıştıran bir sunucuya dağıtıyorsanız, web sunucunuzu yapılandırmak için bir başlangıç noktası olarak aşağıdaki yapılandırma dosyasını kullanabilirsiniz. Büyük olasılıkla, bu dosyanın sunucunuzun yapılandırmasına bağlı olarak özelleştirilmesi gerekecektir. Sunucunuzu yönetme konusunda yardım isterseniz, Laravel Cloud gibi tamamen yönetilen bir Laravel platformu kullanmayı düşünün.

Aşağıdaki yapılandırmada olduğu gibi, web sunucunuzun tüm istekleri uygulamanızın `public/index.php` dosyasına yönlendirdiğinden emin olun. `index.php` dosyasını asla projenizin kök dizinine taşımaya çalışmamalısınız, çünkü uygulamayı proje kökünden sunmak birçok hassas yapılandırma dosyasını genel internete açığa çıkaracaktır.
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### FrankenPHP ile Dağıtım
FrankenPHP, Laravel uygulamalarınızı sunmak için de kullanılabilir. FrankenPHP, Go dilinde yazılmış modern bir PHP uygulama sunucusudur. FrankenPHP kullanarak bir Laravel PHP uygulaması sunmak için, onun `php-server` komutunu çağırmanız yeterlidir:
```bash
frankenphp php-server -r public/
```
FrankenPHP tarafından desteklenen Laravel Octane entegrasyonu, HTTP/3, modern sıkıştırma veya Laravel uygulamalarını bağımsız çalıştırılabilir dosyalar (standalone binaries) olarak paketleme yeteneği gibi daha güçlü özelliklerden yararlanmak için lütfen FrankenPHP'nin Laravel dokümantasyonuna bakın.

### Dizin İzinleri (Directory Permissions)
Laravel'in `bootstrap/cache` ve `storage` dizinlerine yazması gerekecektir, bu nedenle web sunucusu işlem sahibinin (process owner) bu dizinlere yazma iznine sahip olduğundan emin olmalısınız.

## Optimizasyon (Optimization)

Uygulamanızı canlıya dağıtırken, yapılandırmanız (configuration), olaylarınız (events), rotalarınız (routes) ve görünümleriniz (views) dahil olmak üzere önbelleğe alınması gereken çeşitli dosyalar vardır. Laravel, tüm bu dosyaları önbelleğe alacak tek ve kullanışlı bir `optimize` Artisan komutu sağlar. Bu komut tipik olarak uygulamanızın dağıtım sürecinin bir parçası olarak çağrılmalıdır:
```bash
php artisan optimize
```
`optimize:clear` metodu, `optimize` komutu tarafından oluşturulan tüm önbellek dosyalarını ve varsayılan önbellek sürücüsündeki (default cache driver) tüm anahtarları (keys) kaldırmak için kullanılabilir:
```bash
php artisan optimize:clear
```
Aşağıdaki dokümantasyonda, `optimize` komutu tarafından yürütülen ayrıntılı optimizasyon komutlarının her birini tartışacağız.

### Yapılandırmayı Önbelleğe Alma (Caching Configuration)
Uygulamanızı canlıya dağıtırken, dağıtım süreciniz sırasında `config:cache` Artisan komutunu çalıştırdığınızdan emin olmalısınız:
```bash
php artisan config:cache
```
Bu komut, Laravel'in tüm yapılandırma dosyalarını tek bir önbellek dosyasında birleştirecektir; bu, framework'ün yapılandırma değerlerinizi yüklerken dosya sistemine yapması gereken gidiş-geliş (trips) sayısını büyük ölçüde azaltır.

Dağıtım süreciniz sırasında `config:cache` komutunu çalıştırırsanız, `env` fonksiyonunu yalnızca yapılandırma dosyalarınızın içinden çağırdığınızdan emin olmalısınız. Yapılandırma önbelleğe alındıktan sonra `.env` dosyası yüklenmeyecek ve `.env` değişkenleri için `env` fonksiyonuna yapılan tüm çağrılar `null` döndürecektir.

### Olayları Önbelleğe Alma (Caching Events)
Dağıtım süreciniz sırasında uygulamanızın otomatik olarak keşfedilen (auto-discovered) olaydan dinleyiciye (event to listener) eşlemelerini önbelleğe almalısınız. Bu, dağıtım sırasında `event:cache` Artisan komutu çağrılarak gerçekleştirilebilir:
```bash
php artisan event:cache
```

### Rotaları Önbelleğe Alma (Caching Routes)
Birçok rotaya sahip büyük bir uygulama oluşturuyorsanız, dağıtım süreciniz sırasında `route:cache` Artisan komutunu çalıştırdığınızdan emin olmalısınız:
```bash
php artisan route:cache
```
Bu komut, tüm rota kayıtlarınızı önbelleğe alınmış bir dosya içindeki tek bir metot çağrısına indirgeyerek yüzlerce rotayı kaydederken rota kaydının performansını artırır.

### Görünümleri Önbelleğe Alma (Caching Views)
Uygulamanızı canlıya dağıtırken, dağıtım süreciniz sırasında `view:cache` Artisan komutunu çalıştırdığınızdan emin olmalısınız:
```bash
php artisan view:cache
```
Bu komut, tüm Blade görünümlerinizi önceden derleyerek (precompiles) ihtiyaç anında derlenmelerini önler ve bir görünüm döndüren her isteğin performansını artırır.

## Servisleri Yeniden Yükleme (Reloading Services)

Laravel Cloud'a dağıtım yaparken, tüm servislerin sorunsuz bir şekilde yeniden yüklenmesi otomatik olarak yönetildiğinden `reload` komutunu kullanmak gerekli değildir.

Uygulamanızın yeni bir sürümünü dağıttıktan sonra, kuyruk çalışanları (queue workers), Laravel Reverb veya Laravel Octane gibi uzun süre çalışan (long-running) tüm servisler, yeni kodu kullanmak için yeniden yüklenmeli / yeniden başlatılmalıdır. Laravel, bu servisleri sonlandıracak tek bir `reload` Artisan komutu sağlar:
```bash
php artisan reload
```
Laravel Cloud kullanmıyorsanız, yeniden yüklenebilir işlemlerinizin ne zaman çıktığını algılayabilen ve bunları otomatik olarak yeniden başlatan bir işlem monitörünü manuel olarak yapılandırmalısınız.

## Hata Ayıklama Modu (Debug Mode)

`config/app.php` yapılandırma dosyanızdaki `debug` seçeneği, bir hata hakkında kullanıcıya aslında ne kadar bilgi gösterileceğini belirler. Varsayılan olarak bu seçenek, uygulamanızın `.env` dosyasında saklanan `APP_DEBUG` ortam değişkeninin değerine saygı gösterecek şekilde ayarlanmıştır.

Üretim ortamınızda bu değer her zaman `false` olmalıdır. `APP_DEBUG` değişkeni üretimde `true` olarak ayarlanırsa, hassas yapılandırma değerlerini uygulamanızın son kullanıcılarına ifşa etme riskiniz vardır.

## Sağlık Rotası (The Health Route)

Laravel, uygulamanızın durumunu izlemek için kullanılabilecek yerleşik bir sağlık kontrolü (health check) rotası içerir. Üretimde, bu rota uygulamanızın durumunu bir çalışma süresi izleyicisine (uptime monitor), yük dengeleyiciye (load balancer) veya Kubernetes gibi bir orkestrasyon sistemine bildirmek için kullanılabilir.

Varsayılan olarak, sağlık kontrol rotası `/up` adresinde sunulur ve uygulama istisnasız (without exceptions) başlatıldıysa (booted) 200 HTTP yanıtı döndürür. Aksi takdirde, 500 HTTP yanıtı döndürülecektir. Bu rota için URI'yi uygulamanızın `bootstrap/app` dosyasında yapılandırabilirsiniz:
```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up', // Varsayılan
    health: '/status', // Özelleştirilmiş
)
```
Bu rotaya HTTP istekleri yapıldığında, Laravel ayrıca bir `Illuminate\Foundation\Events\DiagnosingHealth` olayı gönderir ve uygulamanızla ilgili ek sağlık kontrolleri yapmanıza olanak tanır. Bu olay için bir dinleyici (listener) içinde, uygulamanızın veritabanını veya önbellek durumunu kontrol edebilirsiniz. Uygulamanızla ilgili bir sorun tespit ederseniz, dinleyiciden sadece bir istisna (exception) fırlatabilirsiniz.

## Laravel Cloud veya Forge ile Dağıtım

#### Laravel Cloud
Laravel için ayarlanmış, tamamen yönetilen, otomatik ölçeklenen bir dağıtım platformu isterseniz, Laravel Cloud'a göz atın. Laravel Cloud, Laravel için yönetilen bilgi işlem (compute), veritabanları, önbellekler ve nesne depolama (object storage) sunan sağlam bir dağıtım platformudur.

Laravel uygulamanızı Cloud'da başlatın ve ölçeklenebilir basitliğe aşık olun. Laravel Cloud, Laravel'in yaratıcıları tarafından framework ile sorunsuz çalışacak şekilde ince ayarlanmıştır, böylece Laravel uygulamalarınızı her zamanki gibi yazmaya devam edebilirsiniz.

#### Laravel Forge
Kendi sunucularınızı yönetmeyi tercih ediyor ancak sağlam bir Laravel uygulaması çalıştırmak için gereken çeşitli servisleri yapılandırma konusunda rahat değilseniz, Laravel Forge, Laravel uygulamaları için bir VPS sunucu yönetim platformudur.

Laravel Forge, DigitalOcean, Linode, AWS ve daha fazlası gibi çeşitli altyapı sağlayıcılarında sunucular oluşturabilir. Ayrıca Forge, Nginx, MySQL, Redis, Memcached, Beanstalk ve daha fazlası gibi sağlam Laravel uygulamaları oluşturmak için gereken tüm araçları kurar ve yönetir.