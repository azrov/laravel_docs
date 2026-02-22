# Laravel 12 Dokümantasyonu: MongoDB

## Giriş

MongoDB, en popüler NoSQL belge odaklı (document-oriented) veritabanlarından biridir. Yüksek yazma yükü (analitik veya IoT için kullanışlıdır) ve yüksek kullanılabilirliği (otomatik hata korumalı (automatic failover) yineleme kümeleri (replica sets) kurması kolaydır) nedeniyle kullanılır. Ayrıca yatay ölçeklenebilirlik (horizontal scalability) için veritabanını kolayca parçalayabilir (shard) ve toplama (aggregation), metin arama veya coğrafi sorgular (geospatial queries) yapmak için güçlü bir sorgu diline sahiptir.

Verileri SQL veritabanları gibi satır veya sütun tablolarında depolamak yerine, MongoDB veritabanındaki her kayıt, verilerin ikili bir temsili (binary representation) olan BSON'da tanımlanan bir belgedir (document). Uygulamalar daha sonra bu bilgileri JSON formatında alabilir. Belgeler, diziler, gömülü belgeler (embedded documents) ve ikili veriler (binary data) dahil olmak üzere çok çeşitli veri türlerini destekler.

Laravel ile MongoDB kullanmadan önce, Composer aracılığıyla `mongodb/laravel-mongodb` paketini kurmanızı ve kullanmanızı öneririz. `laravel-mongodb` paketi resmi olarak MongoDB tarafından sürdürülmektedir ve MongoDB, PHP tarafından MongoDB sürücüsü (driver) aracılığıyla yerel olarak desteklenirken, Laravel MongoDB paketi Eloquent ve diğer Laravel özellikleriyle daha zengin bir entegrasyon sağlar:
```bash
composer require mongodb/laravel-mongodb
```

## Kurulum (Installation)

### MongoDB Sürücüsü (MongoDB Driver)
Bir MongoDB veritabanına bağlanmak için `mongodb` PHP eklentisi (extension) gereklidir. Laravel Herd kullanarak yerel olarak geliştirme yapıyorsanız veya PHP'yi php.new aracılığıyla kurduysanız, bu eklenti sisteminizde zaten kuruludur. Ancak, eklentiyi manuel olarak kurmanız gerekirse, bunu PECL aracılığıyla yapabilirsiniz:
```bash
pecl install mongodb
```
MongoDB PHP eklentisini kurma hakkında daha fazla bilgi için MongoDB PHP eklentisi kurulum talimatlarına bakın.

### MongoDB Sunucusu Başlatma (Starting a MongoDB Server)
MongoDB Community Server, MongoDB'yi yerel olarak çalıştırmak için kullanılabilir ve Windows, macOS, Linux'ta veya bir Docker konteyneri olarak kurulum için mevcuttur. MongoDB'nin nasıl kurulacağını öğrenmek için lütfen resmi MongoDB Community kurulum kılavuzuna bakın.

MongoDB sunucusu için bağlantı dizesi (connection string) `.env` dosyanızda ayarlanabilir:
```
MONGODB_URI="mongodb://localhost:27017"
MONGODB_DATABASE="laravel_app"
```
MongoDB'yi bulutta barındırmak için MongoDB Atlas'ı kullanmayı düşünün. Uygulamanızdan yerel olarak bir MongoDB Atlas kümesine (cluster) erişmek için, kümenin ağ ayarlarında projenin IP Erişim Listesi'ne (IP Access List) kendi IP adresinizi eklemeniz gerekecektir.

MongoDB Atlas için bağlantı dizesi de `.env` dosyanızda ayarlanabilir:
```
MONGODB_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority"
MONGODB_DATABASE="laravel_app"
```

### Laravel MongoDB Paketini Kurma (Install the Laravel MongoDB Package)
Son olarak, Laravel MongoDB paketini kurmak için Composer'ı kullanın:
```bash
composer require mongodb/laravel-mongodb
```
`mongodb` PHP eklentisi kurulu değilse bu paketin kurulumu başarısız olacaktır. PHP yapılandırması CLI ve web sunucusu arasında farklılık gösterebilir, bu nedenle eklentinin her iki yapılandırmada da etkinleştirildiğinden emin olun.

## Yapılandırma (Configuration)

MongoDB bağlantınızı uygulamanızın `config/database.php` yapılandırma dosyası aracılığıyla yapılandırabilirsiniz. Bu dosyada, `mongodb` sürücüsünü kullanan bir `mongodb` bağlantısı ekleyin:
```php
'connections' => [
    'mongodb' => [
        'driver' => 'mongodb',
        'dsn' => env('MONGODB_URI', 'mongodb://localhost:27017'),
        'database' => env('MONGODB_DATABASE', 'laravel_app'),
    ],
],
```

## Özellikler (Features)

Yapılandırmanız tamamlandıktan sonra, uygulamanızda `mongodb` paketini ve veritabanı bağlantısını kullanarak çeşitli güçlü özelliklerden yararlanabilirsiniz:

*   Eloquent'i kullanarak modeller MongoDB koleksiyonlarında saklanabilir. Standart Eloquent özelliklerine ek olarak, Laravel MongoDB paketi gömülü ilişkiler (embedded relationships) gibi ek özellikler sağlar. Paket ayrıca, ham sorgular (raw queries) ve toplama ardışık düzenleri (aggregation pipelines) gibi işlemleri yürütmek için kullanılabilecek MongoDB sürücüsüne doğrudan erişim de sağlar.
*   Sorgu oluşturucuyu (query builder) kullanarak karmaşık sorgular yazın.
*   `mongodb` önbellek sürücüsü, süresi dolmuş önbellek girdilerini otomatik olarak temizlemek için TTL indeksleri gibi MongoDB özelliklerini kullanacak şekilde optimize edilmiştir.
*   `mongodb` kuyruk sürücüsü (queue driver) ile sıraya konmuş işleri (queued jobs) gönderin ve işleyin.
*   Flysystem için GridFS Adaptörü aracılığıyla dosyaları GridFS'de saklama.
*   Bir veritabanı bağlantısı veya Eloquent kullanan çoğu üçüncü taraf paket, MongoDB ile kullanılabilir.

MongoDB ve Laravel'i nasıl kullanacağınızı öğrenmeye devam etmek için MongoDB'nin Hızlı Başlangıç (Quick Start) kılavuzuna bakın.