# Laravel 12 Dokümantasyonu: İstek Yaşam Döngüsü (Request Lifecycle)

## Giriş

"Gerçek dünyada" herhangi bir aracı kullanırken, o aracın nasıl çalıştığını anlarsanız kendinizi daha güvende hissedersiniz. Uygulama geliştirme de farklı değildir. Geliştirme araçlarınızın nasıl işlediğini anladığınızda, onları kullanırken daha rahat ve kendinden emin hissedersiniz.

Bu belgenin amacı, size Laravel framework'ünün nasıl çalıştığına dair iyi ve üst düzey bir genel bakış sunmaktır. Genel framework'ü daha iyi tanıyarak, her şey daha az "sihirli" hissettirecek ve uygulamalarınızı oluştururken daha güvenli olacaksınız. Tüm terimleri hemen anlamazsanız, ümitsizliğe kapılmayın! Sadece neler olup bittiğine dair temel bir kavrayış edinmeye çalışın ve dokümantasyonun diğer bölümlerini keşfettikçe bilginiz artacaktır.

## Yaşam Döngüsüne Genel Bakış (Lifecycle Overview)

### İlk Adımlar (First Steps)
Bir Laravel uygulamasına yapılan tüm isteklerin giriş noktası (entry point) `public/index.php` dosyasıdır. Tüm istekler, web sunucunuzun (Apache / Nginx) yapılandırması tarafından bu dosyaya yönlendirilir. `index.php` dosyası çok fazla kod içermez. Daha ziyade, framework'ün geri kalanını yüklemek için bir başlangıç noktasıdır.

`index.php` dosyası, Composer tarafından oluşturulan otomatik yükleyici (autoloader) tanımını yükler ve ardından `bootstrap/app.php` dosyasından Laravel uygulamasının bir örneğini (instance) alır. Laravel'in kendisi tarafından yapılan ilk işlem, uygulama / servis kabının (service container) bir örneğini oluşturmaktır.

### HTTP / Konsol Çekirdekleri (HTTP / Console Kernels)
Ardından, gelen istek, uygulamaya giren isteğin türüne bağlı olarak, uygulama örneğinin `handleRequest` veya `handleCommand` metotları kullanılarak HTTP çekirdeğine (HTTP kernel) veya konsol çekirdeğine (console kernel) gönderilir. Bu iki çekirdek, tüm isteklerin geçtiği merkezi konum olarak hizmet eder. Şimdilik, sadece `Illuminate\Foundation\Http\Kernel`'in bir örneği olan HTTP çekirdeğine odaklanalım.

HTTP çekirdeği, istek yürütülmeden önce çalıştırılacak bir `bootstrappers` dizisi tanımlar. Bu bootstrapper'lar hata işlemeyi (error handling) yapılandırır, loglamayı (logging) yapılandırır, uygulama ortamını (application environment) algılar ve istek gerçekten işlenmeden önce yapılması gereken diğer görevleri yerine getirir. Tipik olarak, bu sınıflar endişelenmenize gerek olmayan dahili Laravel yapılandırmasını halleder.

HTTP çekirdeği ayrıca isteği uygulamanın ara yazılım (middleware) yığınından geçirmekten sorumludur. Bu ara yazılımlar, HTTP oturumunu (session) okuma ve yazma, uygulamanın bakım modunda (maintenance mode) olup olmadığını belirleme, CSRF token'ını doğrulama ve daha fazlasını yapar. Bunlar hakkında yakında daha fazla konuşacağız.

HTTP çekirdeğinin `handle` metodunun imzası (method signature) oldukça basittir: bir `Request` (İstek) alır ve bir `Response` (Yanıt) döndürür. Çekirdeği, tüm uygulamanızı temsil eden büyük bir kara kutu olarak düşünün. Ona HTTP istekleri besleyin ve o da HTTP yanıtları döndürecektir.

### Servis Sağlayıcılar (Service Providers)
Çekirdek bootstrapping işlemlerinin en önemlilerinden biri, uygulamanız için servis sağlayıcıları (service providers) yüklemektir. Servis sağlayıcılar, veritabanı, kuyruk (queue), doğrulama (validation) ve yönlendirme (routing) bileşenleri gibi framework'ün çeşitli bileşenlerinin tümünü başlatmaktan (bootstrap) sorumludur.

Laravel, bu sağlayıcılar listesinde yineleme yapacak (iterate) ve her birini örnekleyecektir (instantiate). Sağlayıcıları örnekledikten sonra, tüm sağlayıcılarda `register` metodu çağrılacaktır. Ardından, tüm sağlayıcılar kaydedildikten sonra, her sağlayıcıda `boot` metodu çağrılacaktır. Bu sayede servis sağlayıcılar, `boot` metotları yürütüldüğü sırada her kab bağlamasının (container binding) kaydedilmiş ve kullanılabilir olmasına güvenebilirler.

Laravel tarafından sunulan her büyük özellik, temelde bir servis sağlayıcı tarafından başlatılır ve yapılandırılır. Framework tarafından sunulan pek çok özelliği başlattıkları ve yapılandırdıkları için, servis sağlayıcılar tüm Laravel başlatma (bootstrap) sürecinin en önemli unsurudur.

Framework dahili olarak düzinelerce servis sağlayıcı kullanırken, siz de kendi servis sağlayıcılarınızı oluşturma seçeneğine sahipsiniz. Uygulamanız tarafından kullanılan kullanıcı tanımlı veya üçüncü taraf servis sağlayıcıların bir listesini `bootstrap/providers.php` dosyasında bulabilirsiniz.

### Yönlendirme (Routing)
Uygulama başlatıldıktan ve tüm servis sağlayıcılar kaydedildikten sonra, `Request` (İstek) gönderilmek (dispatching) üzere yönlendiriciye (router) devredilecektir. Yönlendirici, isteği bir rotaya (route) veya controller'a gönderecek ve ayrıca rotaya özgü ara yazılımları (middleware) çalıştıracaktır.

Ara yazılımlar (Middleware), uygulamanıza giren HTTP isteklerini filtrelemek veya incelemek için kullanışlı bir mekanizma sağlar. Örneğin, Laravel uygulamanızın kullanıcısının kimlik doğrulamasının yapılıp yapılmadığını doğrulayan bir ara yazılım içerir. Kullanıcının kimlik doğrulaması yapılmamışsa, ara yazılım kullanıcıyı giriş ekranına yönlendirecektir. Ancak, kullanıcının kimlik doğrulaması yapılmışsa, ara yazılım isteğin uygulamanın daha da içine doğru ilerlemesine izin verecektir. Bazı ara yazılımlar, `PreventRequestsDuringMaintenance` gibi uygulama içindeki tüm rotalara atanırken, bazıları yalnızca belirli rotalara veya rota gruplarına atanır. Ara yazılımlar hakkında daha fazla bilgiyi eksiksiz ara yazılım dokümantasyonunu okuyarak öğrenebilirsiniz.

İstek, eşleşen rotaya atanan tüm ara yazılımlardan geçerse, rota veya controller metodu yürütülecek ve rota veya controller metodu tarafından döndürülen yanıt, rotanın ara yazılım zinciri boyunca geri gönderilecektir.

### Tamamlama (Finishing Up)
Rota veya controller metodu bir yanıt döndürdükten sonra, yanıt rotanın ara yazılımlarından geriye doğru dolaşarak uygulamaya giden yanıtı değiştirme veya inceleme şansı verir.

Son olarak, yanıt ara yazılımlardan geri dolaştıktan sonra, HTTP çekirdeğinin `handle` metodu yanıt nesnesini uygulama örneğinin `handleRequest` metoduna döndürür ve bu metod döndürülen yanıt üzerinde `send` metodunu çağırır. `send` metodu, yanıt içeriğini kullanıcının web tarayıcısına gönderir. Artık tüm Laravel istek yaşam döngüsü yolculuğumuzu tamamladık!

## Servis Sağlayıcılara Odaklanın (Focus on Service Providers)

Servis sağlayıcılar, gerçekten bir Laravel uygulamasını başlatmanın (bootstrapping) anahtarıdır. Uygulama örneği oluşturulur, servis sağlayıcılar kaydedilir ve istek başlatılmış uygulamaya devredilir. Gerçekten bu kadar basit!

Bir Laravel uygulamasının servis sağlayıcılar aracılığıyla nasıl oluşturulduğu ve başlatıldığına dair sağlam bir kavrayışa sahip olmak çok değerlidir. Uygulamanızın kullanıcı tanımlı servis sağlayıcıları `app/Providers` dizininde saklanır.

Varsayılan olarak, `AppServiceProvider` oldukça boştur. Bu sağlayıcı, kendi uygulamanızın başlatma (bootstrapping) ve servis kabı bağlamalarını (service container bindings) eklemek için harika bir yerdir. Büyük uygulamalar için, her biri uygulamanız tarafından kullanılan belirli hizmetler için daha ayrıntılı başlatma işlemlerine sahip birkaç servis sağlayıcı oluşturmak isteyebilirsiniz.