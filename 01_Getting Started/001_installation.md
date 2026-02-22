# Laravel 12 Dokümantasyonu (Türkçe)

## Kurulum

### Laravel ile Tanışın

Laravel, anlaşılır ve zarif sözdizimine sahip bir web uygulama framework'üdür. Bir web framework'ü, uygulamanızı oluşturmak için bir yapı ve başlangıç noktası sağlar; biz detaylarla ilgilenirken sizin harika bir şey yaratmaya odaklanmanıza olanak tanır.

Laravel, kapsamlı bağımlılık enjeksiyonu, anlaşılır bir veritabanı soyutlama katmanı, kuyruklar ve zamanlanmış işler, birim ve entegrasyon testleri gibi güçlü özellikler sunarken, harika bir geliştirici deneyimi sağlamayı hedefler.

İster PHP web framework'lerinde yeni olun, ister yılların deneyimine sahip olun, Laravel sizinle birlikte büyüyebilecek bir framework'tür. Bir web geliştiricisi olarak ilk adımlarınızı atmanıza yardımcı oluruz veya uzmanlığınızı bir sonraki seviyeye taşırken size ivme kazandırırız. Neler yapacağınızı görmek için sabırsızlanıyoruz.

### Neden Laravel?

Bir web uygulaması oluştururken kullanabileceğiniz çeşitli araçlar ve framework'ler vardır. Ancak, modern, tam yığın web uygulamaları oluşturmak için en iyi seçimin Laravel olduğuna inanıyoruz.

#### Aşamalı Bir Framework

Laravel'i "aşamalı" bir framework olarak adlandırmayı seviyoruz. Bununla, Laravel'in sizinle birlikte büyüdüğünü kastediyoruz. Web geliştirmeye yeni adım atıyorsanız, Laravel'in geniş dokümantasyon kütüphanesi, kılavuzları ve video eğitimleri, bunalmadan işin temellerini öğrenmenize yardımcı olacaktır.

Kıdemli bir geliştiriciyseniz, Laravel size bağımlılık enjeksiyonu, birim testi, kuyruklar, gerçek zamanlı olaylar ve daha fazlası için sağlam araçlar sunar. Laravel, profesyonel web uygulamaları oluşturmak için ince ayarlanmıştır ve kurumsal iş yüklerini yönetmeye hazırdır.

#### Ölçeklenebilir Bir Framework

Laravel inanılmaz derecede ölçeklenebilir. PHP'nin ölçeklenmeye uygun yapısı ve Laravel'in Redis gibi hızlı, dağıtılmış önbellek sistemleri için yerleşik desteği sayesinde, Laravel ile yatay ölçeklendirme çok kolaydır. Aslında, Laravel uygulamaları ayda yüz milyonlarca isteği kolaylıkla karşılayacak şekilde ölçeklendirilmiştir.

Aşırı ölçeklendirmeye mi ihtiyacınız var? Laravel Cloud gibi platformlar, Laravel uygulamanızı neredeyse sınırsız ölçekte çalıştırmanıza olanak tanır.

#### Yapay Zeka Dostu Bir Framework

Laravel'in belirli kuralları ve iyi tanımlanmış yapısı, onu Cursor ve Claude Code gibi araçları kullanarak yapay zeka destekli geliştirme için ideal bir framework haline getirir. Bir yapay zeka ajanına bir controller (denetleyici) eklemesini söylediğinizde, onu tam olarak nereye koyacağını bilir. Yeni bir migration (veritabanı güncellemesi) gerektiğinde, adlandırma kuralları ve dosya konumları tahmin edilebilirdir. Bu tutarlılık, daha esnek framework'lerde yapay zeka araçlarını sıklıkla zorlayan tahmin yürütmeyi ortadan kaldırır.

Dosya organizasyonunun ötesinde, Laravel'in anlaşılır sözdizimi ve kapsamlı dokümantasyonu, yapay zeka ajanlarına doğru, deyimsel kod üretmeleri için ihtiyaç duydukları bağlamı sağlar. Eloquent ilişkileri, form istekleri ve ara yazılımlar gibi özellikler, ajanların güvenilir bir şekilde anlayıp kopyalayabileceği kalıpları izler. Sonuç, deneyimli bir Laravel geliştiricisi tarafından yazılmış gibi görünen, genel PHP parçacıklarından bir araya getirilmemiş, yapay zeka tarafından oluşturulmuş koddur.

Laravel'in yapay zeka destekli geliştirme için neden mükemmel bir seçim olduğu hakkında daha fazla bilgi edinmek için, ajanik geliştirme ile ilgili dokümantasyonumuza göz atın.

#### Topluluk Framework'ü

Laravel, PHP ekosistemindeki en iyi paketleri birleştirerek mevcut en sağlam ve geliştirici dostu framework'ü sunar. Ayrıca, dünyanın dört bir yanından binlerce yetenekli geliştirici framework'e katkıda bulunmuştur. Kim bilir, belki siz de bir Laravel katkıcısı olabilirsiniz.

## Laravel Uygulaması Oluşturma

### PHP ve Laravel Kurucusunu Yükleme

İlk Laravel uygulamanızı oluşturmadan önce, yerel makinenizde PHP, Composer ve Laravel kurucusunun yüklü olduğundan emin olun. Ayrıca, uygulamanızın ön yüz (frontend) varlıklarını derleyebilmek için Node ve NPM veya Bun yüklemelisiniz.

Eğer yerel makinenizde PHP ve Composer yüklü değilse, aşağıdaki komutlar macOS, Windows veya Linux'a PHP, Composer ve Laravel kurucusunu yükleyecektir:

**macOS için:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

**Windows için (yönetici olarak çalıştırın):**
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

**Linux için:**
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

Yukarıdaki komutlardan birini çalıştırdıktan sonra, terminal oturumunuzu yeniden başlatmalısınız. PHP, Composer ve Laravel kurucusunu `php.new` üzerinden yükledikten sonra güncellemek için, terminalde komutu yeniden çalıştırabilirsiniz.

Eğer zaten PHP ve Composer'ınız yüklüyse, Laravel kurucusunu Composer aracılığıyla yükleyebilirsiniz:
```bash
composer global require laravel/installer
```

Tam özellikli, grafiksel bir PHP kurulum ve yönetim deneyimi için Laravel Herd'e göz atın.

### Uygulama Oluşturma

PHP, Composer ve Laravel kurucusunu yükledikten sonra, yeni bir Laravel uygulaması oluşturmaya hazırsınız. Laravel kurucusu, tercih ettiğiniz test framework'ünü, veritabanını ve başlangıç kitini (starter kit) seçmeniz için size sorular soracaktır:
```bash
laravel new example-app
```

Uygulama oluşturulduktan sonra, Laravel'in yerel geliştirme sunucusunu, kuyruk işçisini (queue worker) ve Vite geliştirme sunucusunu `dev` Composer betiğini kullanarak başlatabilirsiniz:
```bash
cd example-app
npm install && npm run build
composer run dev
```

Geliştirme sunucusunu başlattıktan sonra, uygulamanıza web tarayıcınızdan `http://localhost:8000` adresinden erişebilirsiniz. Sonraki adım, Laravel ekosisteminde ilerlemeye başlamak. Elbette, bir veritabanı yapılandırmak da isteyebilirsiniz.

Laravel uygulamanızı geliştirirken hızlı bir başlangıç yapmak isterseniz, başlangıç kitlerimizden (starter kits) birini kullanmayı düşünün. Laravel'in başlangıç kitleri, yeni Laravel uygulamanız için arka uç (backend) ve ön uç (frontend) kimlik doğrulama iskeleti sağlar.

## İlk Yapılandırma

Laravel framework'üne ait tüm yapılandırma dosyaları `config` dizininde bulunur. Her seçenek belgelenmiştir, bu nedenle dosyaları incelemekten ve size sunulan seçeneklere aşina olmaktan çekinmeyin.

Laravel kutudan çıktığı gibi neredeyse hiç ek yapılandırma gerektirmez. Geliştirmeye başlamakta özgürsünüz! Ancak, `config/app.php` dosyasını ve dokümantasyonunu gözden geçirmek isteyebilirsiniz. Bu dosya, uygulamanıza göre değiştirmek isteyebileceğiniz `url` ve `locale` gibi birkaç seçenek içerir.

### Ortam Tabanlı Yapılandırma

Laravel'in birçok yapılandırma seçeneği değeri, uygulamanızın yerel makinenizde mi yoksa bir üretim web sunucusunda mı çalıştığına bağlı olarak değişebileceğinden, birçok önemli yapılandırma değeri, uygulamanızın kök dizininde bulunan `.env` dosyası kullanılarak tanımlanır.

`.env` dosyanız, uygulamanızı kullanan her geliştirici/sunucu farklı bir ortam yapılandırması gerektirebileceğinden, uygulamanızın kaynak kontrolüne (source control) eklenmemelidir. Ayrıca, bir saldırgan kaynak kontrol deponuza erişim sağlarsa, herhangi bir hassas kimlik bilgisi (credentials) ifşa olacağından bu bir güvenlik riski oluşturur.

`.env` dosyası ve ortam tabanlı yapılandırma hakkında daha fazla bilgi için tam yapılandırma dokümantasyonuna göz atın.

### Veritabanları ve Migration'lar (Veritabanı Güncellemeleri)

Artık Laravel uygulamanızı oluşturduğunuza göre, muhtemelen bir veritabanında bazı veriler depolamak istersiniz. Varsayılan olarak, uygulamanızın `.env` yapılandırma dosyası, Laravel'in bir SQLite veritabanı ile etkileşime gireceğini belirtir.

Uygulamanın oluşturulması sırasında, Laravel sizin için `database/database.sqlite` dosyasını oluşturdu ve uygulamanın veritabanı tablolarını oluşturmak için gerekli migration'ları çalıştırdı.

MySQL veya PostgreSQL gibi başka bir veritabanı sürücüsü kullanmayı tercih ederseniz, uygun veritabanını kullanmak için `.env` yapılandırma dosyanızı güncelleyebilirsiniz. Örneğin, MySQL kullanmak istiyorsanız, `.env` yapılandırma dosyanızdaki `DB_*` değişkenlerini aşağıdaki gibi güncelleyin:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

SQLite dışında bir veritabanı kullanmayı seçerseniz, veritabanını oluşturmanız ve uygulamanızın veritabanı migration'larını çalıştırmanız gerekecektir:
```bash
php artisan migrate
```

macOS veya Windows üzerinde geliştirme yapıyorsanız ve yerel olarak MySQL, PostgreSQL veya Redis kurmanız gerekiyorsa, Herd Pro veya DBngin'i kullanmayı düşünün.

### Dizin Yapılandırması

Laravel her zaman web sunucunuz için yapılandırılmış "web dizininin" kökünden sunulmalıdır. Bir Laravel uygulamasını "web dizininin" bir alt dizininden sunmaya çalışmamalısınız. Bunu yapmaya çalışmak, uygulamanızda bulunan hassas dosyaları açığa çıkarabilir.

## Herd Kullanarak Kurulum

Laravel Herd, macOS ve Windows için son derece hızlı, yerel bir Laravel ve PHP geliştirme ortamıdır. Herd, Laravel geliştirmeye başlamak için ihtiyacınız olan her şeyi (PHP ve Nginx dahil) içerir.

Herd'i yükledikten sonra, Laravel ile geliştirmeye başlamaya hazırsınız. Herd, `php`, `composer`, `laravel`, `expose`, `node`, `npm` ve `nvm` için komut satırı araçları içerir.

Herd Pro, Herd'i yerel MySQL, Postgres ve Redis veritabanları oluşturup yönetme yeteneği, yerel posta görüntüleme ve log izleme gibi ek güçlü özelliklerle zenginleştirir.

### macOS'te Herd

macOS üzerinde geliştirme yapıyorsanız, Herd web sitesinden Herd kurucusunu indirebilirsiniz. Kurucu, otomatik olarak PHP'nin en son sürümünü indirir ve Mac'inizi Nginx'i her zaman arka planda çalıştıracak şekilde yapılandırır.

macOS için Herd, "park edilmiş" dizinleri desteklemek için dnsmasq kullanır. Park edilmiş bir dizindeki herhangi bir Laravel uygulaması, Herd tarafından otomatik olarak sunulacaktır. Varsayılan olarak Herd, `~/Herd` konumunda bir park edilmiş dizin oluşturur ve bu dizindeki herhangi bir Laravel uygulamasına, dizin adını kullanarak `.test` alan adı üzerinden erişebilirsiniz.

Herd'i yükledikten sonra, yeni bir Laravel uygulaması oluşturmanın en hızlı yolu, Herd ile birlikte gelen Laravel CLI'yı kullanmaktır:
```bash
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

Elbette, park edilmiş dizinlerinizi ve diğer PHP ayarlarınızı her zaman sistem tepsisindeki Herd menüsünden açılabilen Herd'in kullanıcı arayüzü (UI) aracılığıyla yönetebilirsiniz.

Herd hakkında daha fazla bilgiyi Herd dokümantasyonuna bakarak öğrenebilirsiniz.

### Windows'ta Herd

Windows için Herd kurucusunu Herd web sitesinden indirebilirsiniz. Kurulum tamamlandıktan sonra, Herd'i başlatarak hazırlık sürecini tamamlayabilir ve Herd kullanıcı arayüzüne ilk kez erişebilirsiniz.

Herd kullanıcı arayüzüne, Herd'in sistem tepsisi simgesine sol tıklayarak erişilebilir. Sağ tıklama, günlük olarak ihtiyacınız olan tüm araçlara erişim sağlayan hızlı menüyü açar.

Kurulum sırasında Herd, ana dizininizde `%USERPROFILE%\Herd` konumunda bir "park edilmiş" dizin oluşturur. Park edilmiş bir dizindeki herhangi bir Laravel uygulaması, Herd tarafından otomatik olarak sunulacaktır ve bu dizindeki herhangi bir Laravel uygulamasına, dizin adını kullanarak `.test` alan adı üzerinden erişebilirsiniz.

Herd'i yükledikten sonra, yeni bir Laravel uygulaması oluşturmanın en hızlı yolu, Herd ile birlikte gelen Laravel CLI'yı kullanmaktır. Başlamak için Powershell'i açın ve aşağıdaki komutları çalıştırın:
```powershell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

Herd hakkında daha fazla bilgiyi Windows için Herd dokümantasyonuna bakarak öğrenebilirsiniz.

## IDE Desteği

Laravel uygulamaları geliştirirken istediğiniz herhangi bir kod düzenleyiciyi (code editor) kullanmakta özgürsünüz. Hafif ve genişletilebilir düzenleyiciler arıyorsanız, resmi Laravel VS Code Eklentisi ile birleştirilmiş VS Code veya Cursor, sözdizimi vurgulama, snippet'ler, artisan komut entegrasyonu ve Eloquent modelleri, rotalar, ara yazılımlar, varlıklar, yapılandırma ve Inertia.js için akıllı otomatik tamamlama gibi mükemmel Laravel desteği sunar.

Laravel için kapsamlı ve sağlam destek için, bir JetBrains IDE'si olan PhpStorm'a göz atın. PhpStorm'in yerleşik Laravel framework desteği, Blade şablonları, Eloquent modelleri, rotalar, görünümler, çeviriler ve bileşenler için akıllı otomatik tamamlama, ayrıca Laravel projelerinde güçlü kod oluşturma ve gezinme özelliklerini içerir.

Bulut tabanlı bir geliştirme deneyimi arayanlar için Firebase Studio, doğrudan tarayıcınızda Laravel ile oluşturmaya anında erişim sağlar. Sıfır kurulum gereksinimi ile Firebase Studio, herhangi bir cihazdan Laravel uygulamaları oluşturmaya başlamayı kolaylaştırır.

## Laravel ve Yapay Zeka (YZ)

Laravel Boost, YZ kodlama ajanları ile Laravel uygulamaları arasında köprü kuran güçlü bir araçtır. Boost, YZ ajanlarına Laravel'e özgü bağlam, araçlar ve yönergeler sağlayarak, Laravel kurallarına uyan daha doğru, sürüme özel kod üretmelerini sağlar.

Boost'u Laravel uygulamanıza yüklediğinizde, YZ ajanları hangi paketleri kullandığınızı bilme, veritabanınızı sorgulama, Laravel dokümantasyonunda arama yapma, tarayıcı loglarını okuma, testler oluşturma ve Tinker aracılığıyla kod çalıştırma gibi 15'in üzerinde özel araca erişim kazanır.

Ayrıca Boost, YZ ajanlarına, yüklü paket sürümlerinize özgü, vektörleştirilmiş 17.000'den fazla Laravel ekosistemi dokümantasyon parçasına erişim sağlar. Bu, ajanların projenizin kullandığı tam sürümlere yönelik rehberlik sağlayabileceği anlamına gelir.

Boost ayrıca, ajanların framework kurallarını takip etmesine, uygun testler yazmasına ve Laravel kodu oluştururken yaygın tuzaklardan kaçınmasına yardımcı olan Laravel tarafından bakımı yapılan YZ yönergelerini de içerir.

### Laravel Boost'u Yükleme

Boost, PHP 8.1 veya üstünü çalıştıran Laravel 10, 11 ve 12 uygulamalarına yüklenebilir. Başlamak için Boost'u bir geliştirme bağımlılığı olarak yükleyin:
```bash
composer require laravel/boost --dev
```

Yüklendikten sonra, etkileşimli kurucuyu çalıştırın:
```bash
php artisan boost:install
```

Kurucu, IDE'nizi ve YZ ajanlarınızı otomatik olarak algılayacak ve projeniz için anlamlı olan özellikleri tercih etmenize olanak tanıyacaktır. Boost, mevcut proje kurallarına saygı duyar ve varsayılan olarak görüşe dayalı stil kurallarını dayatmaz.

Boost hakkında daha fazla bilgi edinmek için GitHub'daki Laravel Boost deposuna göz atın.

#### Özel YZ Yönergeleri Ekleme

Laravel Boost'u kendi özel YZ yönergelerinizle zenginleştirmek için, uygulamanızın `.ai/guidelines/*` dizinine `.blade.php` veya `.md` dosyaları ekleyin. Bu dosyalar, `boost:install` komutunu çalıştırdığınızda Laravel Boost'un yönergelerine otomatik olarak dahil edilecektir.

## Sonraki Adımlar

Artık Laravel uygulamanızı oluşturduğunuza göre, bundan sonra ne yapacağınızı merak ediyor olabilirsiniz.