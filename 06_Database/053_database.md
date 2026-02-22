# Laravel 12 Dokümantasyonu: Veritabanı: Başlarken (Database: Getting Started)

## Giriş

Neredeyse her modern web uygulaması bir veritabanıyla etkileşime girer. Laravel, ham SQL, akıcı bir sorgu oluşturucu (fluent query builder) ve Eloquent ORM kullanarak, desteklenen çeşitli veritabanlarıyla etkileşim kurmayı son derece basit hale getirir. Şu anda Laravel, beş veritabanı için birinci taraf (first-party) destek sağlamaktadır:

*   MariaDB
*   MySQL
*   PostgreSQL
*   SQLite
*   SQL Server

Ek olarak, MongoDB, MongoDB tarafından resmi olarak sürdürülen `mongodb/laravel-mongodb` paketi aracılığıyla desteklenmektedir. Daha fazla bilgi için Laravel MongoDB dokümantasyonuna göz atın.

### Yapılandırma (Configuration)
Laravel'in veritabanı servislerinin yapılandırması, uygulamanızın `config/database.php` yapılandırma dosyasında bulunur. Bu dosyada, tüm veritabanı bağlantılarınızı (database connections) tanımlayabilir ve varsayılan olarak hangi bağlantının kullanılması gerektiğini belirtebilirsiniz. Bu dosyadaki yapılandırma seçeneklerinin çoğu, uygulamanızın ortam değişkenlerinin (environment variables) değerleri tarafından yönlendirilir. Laravel tarafından desteklenen veritabanı sistemlerinin çoğu için örnekler bu dosyada sağlanmıştır.

Varsayılan olarak, Laravel'in örnek ortam yapılandırması, yerel makinenizde Laravel uygulamaları geliştirmek için bir Docker yapılandırması olan Laravel Sail ile kullanıma hazırdır. Ancak, yerel veritabanınız için gerektiği gibi veritabanı yapılandırmanızı değiştirmekte özgürsünüz.

#### SQLite Yapılandırması (SQLite Configuration)
SQLite veritabanları, dosya sisteminizde tek bir dosya içinde bulunur. Terminalinizde `touch` komutunu kullanarak yeni bir SQLite veritabanı oluşturabilirsiniz: `touch database/database.sqlite`. Veritabanı oluşturulduktan sonra, `DB_DATABASE` ortam değişkenine veritabanının mutlak yolunu (absolute path) koyarak ortam değişkenlerinizi bu veritabanını işaret edecek şekilde kolayca yapılandırabilirsiniz:
```
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```
Varsayılan olarak, yabancı anahtar kısıtlamaları (foreign key constraints) SQLite bağlantıları için etkindir. Bunları devre dışı bırakmak isterseniz, `DB_FOREIGN_KEYS` ortam değişkenini `false` olarak ayarlamalısınız:
```
DB_FOREIGN_KEYS=false
```
Laravel uygulamanızı oluşturmak için Laravel kurucusunu (installer) kullanırsanız ve veritabanı olarak SQLite'ı seçerseniz, Laravel otomatik olarak bir `database/database.sqlite` dosyası oluşturacak ve sizin için varsayılan veritabanı migration'larını çalıştıracaktır.

#### Microsoft SQL Server Yapılandırması (Microsoft SQL Server Configuration)
Bir Microsoft SQL Server veritabanı kullanmak için, `sqlsrv` ve `pdo_sqlsrv` PHP eklentilerinin yanı sıra Microsoft SQL ODBC sürücüsü gibi gerektirebilecekleri bağımlılıkların kurulu olduğundan emin olmalısınız.

#### URL'leri Kullanarak Yapılandırma (Configuration Using URLs)
Tipik olarak, veritabanı bağlantıları `host`, `database`, `username`, `password` vb. gibi birden çok yapılandırma değeri kullanılarak yapılandırılır. Bu yapılandırma değerlerinin her birinin kendine ait karşılık gelen bir ortam değişkeni vardır. Bu, bir üretim sunucusunda veritabanı bağlantı bilgilerinizi yapılandırırken birden çok ortam değişkenini yönetmeniz gerektiği anlamına gelir.

AWS ve Heroku gibi bazı yönetilen veritabanı sağlayıcıları, veritabanı için tüm bağlantı bilgilerini tek bir dizede içeren tek bir veritabanı "URL'si" sağlar. Örnek bir veritabanı URL'si şuna benzer:
```
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```
Bu URL'ler tipik olarak standart bir şema kuralını takip eder:
```
driver://username:password@host:port/database?options
```
Kolaylık olması açısından Laravel, veritabanınızı birden çok yapılandırma seçeneğiyle yapılandırmaya bir alternatif olarak bu URL'leri destekler. `url` (veya karşılık gelen `DB_URL` ortam değişkeni) yapılandırma seçeneği mevcutsa, veritabanı bağlantısını ve kimlik bilgisi bilgilerini çıkarmak için kullanılacaktır.

### Okuma ve Yazma Bağlantıları (Read and Write Connections)
Bazen SELECT ifadeleri için bir veritabanı bağlantısı, INSERT, UPDATE ve DELETE ifadeleri için ise başka bir veritabanı bağlantısı kullanmak isteyebilirsiniz. Laravel bunu çok kolaylaştırır ve ister ham sorgular, ister sorgu oluşturucu veya Eloquent ORM kullanıyor olun, uygun bağlantılar her zaman kullanılacaktır.

Okuma / yazma bağlantılarının nasıl yapılandırılması gerektiğini görmek için şu örneğe bakalım:
```php
'mysql' => [
    'driver' => 'mysql',

    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '192.168.1.3',
        ],
    ],
    'sticky' => true,

    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        (PHP_VERSION_ID >= 80500 ? \Pdo\Mysql::ATTR_SSL_CA : \PDO::MYSQL_ATTR_SSL_CA) => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```
Yapılandırma dizisine üç anahtarın eklendiğine dikkat edin: `read`, `write` ve `sticky`. `read` ve `write` anahtarları, tek bir anahtar içeren dizi değerlerine sahiptir: `host`. Okuma ve yazma bağlantıları için veritabanı seçeneklerinin geri kalanı, ana `mysql` yapılandırma dizisinden birleştirilecektir (merged).

Yalnızca ana `mysql` dizisindeki değerleri geçersiz kılmak (override) istiyorsanız, öğeleri `read` ve `write` dizilerine koymanız yeterlidir. Yani, bu durumda, "okuma" bağlantısı için ana bilgisayar olarak `192.168.1.1` kullanılırken, "yazma" bağlantısı için `192.168.1.3` kullanılacaktır. Veritabanı kimlik bilgileri, ön ek (prefix), karakter seti ve ana `mysql` dizisindeki diğer tüm seçenekler her iki bağlantı arasında paylaşılacaktır. Ana bilgisayar yapılandırma dizisinde birden çok değer olduğunda, her istek için rastgele bir veritabanı ana bilgisayarı seçilecektir.

#### `sticky` Seçeneği (The sticky Option)
`sticky` seçeneği, geçerli istek döngüsü sırasında veritabanına yazılan kayıtların hemen okunmasına izin vermek için kullanılabilen isteğe bağlı bir değerdir. `sticky` seçeneği etkinleştirilmişse ve geçerli istek döngüsü sırasında veritabanına karşı bir "yazma" işlemi gerçekleştirilmişse, sonraki herhangi bir "okuma" işlemi "yazma" bağlantısını kullanacaktır. Bu, istek döngüsü sırasında yazılan herhangi bir verinin aynı istek sırasında veritabanından hemen okunabilmesini sağlar. Bunun uygulamanız için istenen davranış olup olmadığına karar vermek size kalmıştır.

## SQL Sorguları Çalıştırma (Running SQL Queries)

Veritabanı bağlantınızı yapılandırdıktan sonra, `DB` facade'ını kullanarak sorgular çalıştırabilirsiniz. `DB` facade'ı her sorgu türü için metotlar sağlar: `select`, `update`, `insert`, `delete` ve `statement`.

#### Select Sorgusu Çalıştırma (Running a Select Query)
Temel bir SELECT sorgusu çalıştırmak için `DB` facade'ındaki `select` metodunu kullanabilirsiniz:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Uygulamanın tüm kullanıcılarının bir listesini göster.
     */
    public function index(): View
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```
`select` metoduna iletilen ilk argüman SQL sorgusudur, ikinci argüman ise sorguya bağlanması gereken herhangi bir parametre bağlamasıdır (parameter bindings). Tipik olarak bunlar, where cümlesi kısıtlamalarının değerleridir. Parametre bağlama, SQL enjeksiyonuna (SQL injection) karşı koruma sağlar.

`select` metodu her zaman bir sonuç dizisi döndürecektir. Dizi içindeki her sonuç, veritabanından bir kaydı temsil eden bir PHP `stdClass` nesnesi olacaktır:
```php
use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

#### Skaler (Scalar) Değerleri Seçme (Selecting Scalar Values)
Bazen veritabanı sorgunuz tek bir skaler değerle sonuçlanabilir. Laravel, sorgunun skaler sonucunu bir kayıt nesnesinden almanız gerekmeksizin, bu değeri doğrudan `scalar` metodunu kullanarak almanıza izin verir:
```php
$burgers = DB::scalar(
    "select count(case when food = 'burger' then 1 end) as burgers from menu"
);
```

#### Birden Çok Sonuç Kümesi Seçme (Selecting Multiple Result Sets)
Uygulamanız, birden çok sonuç kümesi döndüren saklı yordamları (stored procedures) çağırıyorsa, saklı yordam tarafından döndürülen tüm sonuç kümelerini almak için `selectResultSets` metodunu kullanabilirsiniz:
```php
[$options, $notifications] = DB::selectResultSets(
    "CALL get_user_options_and_notifications(?)", $request->user()->id
);
```

#### Adlandırılmış Bağlamaları Kullanma (Using Named Bindings)
Parametre bağlamalarınızı temsil etmek için `?` kullanmak yerine, adlandırılmış bağlamalar (named bindings) kullanarak bir sorgu çalıştırabilirsiniz:
```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

#### Insert İfadesi Çalıştırma (Running an Insert Statement)
Bir `insert` ifadesi çalıştırmak için `DB` facade'ındaki `insert` metodunu kullanabilirsiniz. `select` gibi, bu metod da ilk argüman olarak SQL sorgusunu ve ikinci argüman olarak bağlamaları kabul eder:
```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);
```

#### Update İfadesi Çalıştırma (Running an Update Statement)
`update` metodu, veritabanındaki mevcut kayıtları güncellemek için kullanılmalıdır. İfadeden etkilenen satır sayısı metod tarafından döndürülür:
```php
use Illuminate\Support\Facades\DB;

$affected = DB::update(
    'update users set votes = 100 where name = ?',
    ['Anita']
);
```

#### Delete İfadesi Çalıştırma (Running a Delete Statement)
`delete` metodu, veritabanından kayıtları silmek için kullanılmalıdır. `update` gibi, etkilenen satır sayısı metod tarafından döndürülecektir:
```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

#### Genel Bir İfade Çalıştırma (Running a General Statement)
Bazı veritabanı ifadeleri herhangi bir değer döndürmez. Bu tür işlemler için `DB` facade'ındaki `statement` metodunu kullanabilirsiniz:
```php
DB::statement('drop table users');
```

#### Hazırlıksız (Unprepared) Bir İfade Çalıştırma (Running an Unprepared Statement)
Bazen herhangi bir değer bağlamadan bir SQL ifadesi çalıştırmak isteyebilirsiniz. Bunu başarmak için `DB` facade'ının `unprepared` metodunu kullanabilirsiniz:
```php
DB::unprepared('update users set votes = 100 where name = "Dries"');
```
Hazırlıksız ifadeler parametreleri bağlamadığından, SQL enjeksiyonuna karşı savunmasız olabilirler. Hazırlıksız bir ifade içinde asla kullanıcı kontrollü değerlere izin vermemelisiniz.

#### Örtük Commit'ler (Implicit Commits)
İşlemler (transactions) içinde `DB` facade'ının `statement` ve `unprepared` metotlarını kullanırken, örtük commit'lere (implicit commits) neden olan ifadelerden kaçınmaya dikkat etmelisiniz. Bu ifadeler, veritabanı motorunun tüm işlemi dolaylı olarak (indirectly) commit etmesine neden olarak Laravel'in veritabanının işlem seviyesinden habersiz kalmasına yol açar. Böyle bir ifadeye örnek olarak bir veritabanı tablosu oluşturmak verilebilir:
```php
DB::unprepared('create table a (col varchar(1) null)');
```
Örtük commit'leri tetikleyen tüm ifadelerin listesi için MySQL kılavuzuna bakın.

### Birden Çok Veritabanı Bağlantısı Kullanma (Using Multiple Database Connections)
Uygulamanız `config/database.php` yapılandırma dosyanızda birden çok bağlantı tanımlıyorsa, `DB` facade'ı tarafından sağlanan `connection` metodu aracılığıyla her bir bağlantıya erişebilirsiniz. `connection` metoduna iletilen bağlantı adı, `config/database.php` yapılandırma dosyanızda listelenen bağlantılardan birine veya `config` yardımcısı kullanılarak çalışma zamanında yapılandırılan bir bağlantıya karşılık gelmelidir:
```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(/* ... */);
```
Bir bağlantı örneği üzerindeki `getPdo` metodunu kullanarak bir bağlantının ham, temeldeki PDO örneğine (raw, underlying PDO instance) erişebilirsiniz:
```php
$pdo = DB::connection()->getPdo();
```

### Sorgu Olaylarını Dinleme (Listening for Query Events)
Uygulamanız tarafından çalıştırılan her SQL sorgusu için çağrılacak bir closure belirtmek isterseniz, `DB` facade'ının `listen` metodunu kullanabilirsiniz. Bu metod, sorguları günlüğe kaydetmek veya hata ayıklama için kullanışlı olabilir. Sorgu dinleyici closure'ınızı bir servis sağlayıcının (service provider) `boot` metodunda kaydedebilirsiniz:
```php
<?php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini kaydet.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        DB::listen(function (QueryExecuted $query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
            // $query->toRawSql();
        });
    }
}
```

### Kümülatif Sorgu Süresini İzleme (Monitoring Cumulative Query Time)
Modern web uygulamalarının yaygın bir performans darboğazı, veritabanlarını sorgulamak için harcadıkları süredir. Neyse ki Laravel, tek bir istek sırasında veritabanını sorgulamak için çok fazla zaman harcadığında, seçtiğiniz bir closure veya geri çağrıyı (callback) çağırabilir. Başlamak için, `whenQueryingForLongerThan` metoduna bir sorgu süresi eşiği (milisaniye cinsinden) ve bir closure sağlayın. Bu metodu bir servis sağlayıcının `boot` metodunda çağırabilirsiniz:
```php
<?php

namespace App\Providers;

use Illuminate\Database\Connection;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Events\QueryExecuted;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini kaydet.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // Geliştirme ekibine bildir...
        });
    }
}
```

## Veritabanı İşlemleri (Database Transactions)

`DB` facade'ı tarafından sağlanan `transaction` metodunu, bir dizi işlemi bir veritabanı işlemi (database transaction) içinde çalıştırmak için kullanabilirsiniz. İşlem closure'ı içinde bir istisna (exception) fırlatılırsa, işlem otomatik olarak geri alınacak (rolled back) ve istisna yeniden fırlatılacaktır. Closure başarıyla yürütülürse, işlem otomatik olarak onaylanacaktır (committed). `transaction` metodunu kullanırken manuel olarak geri alma veya onaylama konusunda endişelenmenize gerek yoktur:
```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
});
```

#### Kilitlenmeleri (Deadlocks) Ele Alma
`transaction` metodu, kilitlenmeleri (deadlocks) ele almak için isteğe bağlı ikinci bir argüman kabul eder. Bu argüman, bir kilitlenme durumunda işlemin kaç kez yeniden denenmesi gerektiğini belirtir. Bu denemelerin tükenmesi durumunda bir istisna fırlatılacaktır:
```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
}, 5);
```

#### Manüel İşlemler (Manually Using Transactions)
İşlemleri manuel olarak başlatmak ve geri almak / onaylamak isterseniz, `DB` facade'ı tarafından sağlanan `beginTransaction` metodunu kullanabilirsiniz:
```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();
```
`rollBack` metodu ile işlemi geri alabilirsiniz:
```php
DB::rollBack();
```
Son olarak, `commit` metodu ile işlemi onaylayabilirsiniz:
```php
DB::commit();
```
`DB` facade'ının transaction metotları, transaction seviyesini kontrol eden bağlantı ve transaction sayacını da sorgulayabilir; bu, özellikle iç içe işlemler (nested transactions) için kullanışlıdır:
```php
// Transaction başlatıldı mı?
DB::transactionLevel();

// Mevcut transaction'ın bağlantısını al...
DB::getTransactionConnection();
```