# Laravel 12 Dokümantasyonu: Veritabanı: Sayfalama (Database: Pagination)

## Giriş

Diğer framework'lerde sayfalama (pagination) çok zahmetli olabilir. Laravel'in sayfalama yaklaşımının size ferahlatıcı bir soluk getireceğini umuyoruz. Laravel'in sayfalayıcısı (paginator), sorgu oluşturucu (query builder) ve Eloquent ORM ile entegredir ve sıfır yapılandırma ile veritabanı kayıtlarının kullanışlı, kolay kullanımlı sayfalamasını sağlar.

Varsayılan olarak, sayfalayıcı tarafından oluşturulan HTML, Tailwind CSS framework'ü ile uyumludur; ancak, Bootstrap sayfalama desteği de mevcuttur.

#### Tailwind
Laravel'in varsayılan Tailwind sayfalama görünümlerini Tailwind 4.x ile kullanıyorsanız, uygulamanızın `resources/css/app.css` dosyası, Laravel'in sayfalama görünümlerini `@source` olarak gösterecek şekilde zaten uygun şekilde yapılandırılmış olacaktır:
```css
@import 'tailwindcss';

@source '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
```

## Temel Kullanım (Basic Usage)

### Sorgu Oluşturucu Sonuçlarını Sayfalama (Paginating Query Builder Results)
Öğeleri sayfalamanın birkaç yolu vardır. En basiti, sorgu oluşturucu veya bir Eloquent sorgusunda `paginate` metodunu kullanmaktır. `paginate` metodu, kullanıcı tarafından görüntülenen geçerli sayfaya göre sorgunun "limit" ve "offset" değerlerini ayarlamayı otomatik olarak halleder. Varsayılan olarak, geçerli sayfa, HTTP isteğindeki `page` sorgu dizesi argümanının değeriyle algılanır. Bu değer Laravel tarafından otomatik olarak algılanır ve ayrıca sayfalayıcı tarafından oluşturulan bağlantılara otomatik olarak eklenir.

Bu örnekte, `paginate` metoduna iletilen tek argüman, "sayfa başına" görüntülemek istediğiniz öğe sayısıdır. Bu durumda, sayfa başına 15 öğe görüntülemek istediğimizi belirtelim:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Tüm uygulama kullanıcılarını göster.
     */
    public function index(): View
    {
        return view('user.index', [
            'users' => DB::table('users')->paginate(15)
        ]);
    }
}
```

#### Basit Sayfalama (Simple Pagination)
`paginate` metodu, kayıtları veritabanından almadan önce sorguyla eşleşen toplam kayıt sayısını sayar. Bu, sayfalayıcının toplamda kaç sayfa kayıt olduğunu bilmesi için yapılır. Ancak, uygulamanızın kullanıcı arayüzünde (UI) toplam sayfa sayısını göstermeyi planlamıyorsanız, kayıt sayısı sorgusu gereksizdir.

Bu nedenle, uygulamanızın kullanıcı arayüzünde yalnızca basit "Sonraki" ve "Önceki" bağlantılarını görüntülemeniz gerekiyorsa, tek, verimli bir sorgu gerçekleştirmek için `simplePaginate` metodunu kullanabilirsiniz:
```php
$users = DB::table('users')->simplePaginate(15);
```

### Eloquent Sonuçlarını Sayfalama (Paginating Eloquent Results)
Eloquent sorgularını da sayfalayabilirsiniz. Bu örnekte, `App\Models\User` modelini sayfalayacağız ve sayfa başına 15 kayıt görüntülemeyi planladığımızı belirteceğiz. Gördüğünüz gibi, sözdizimi sorgu oluşturucu sonuçlarını sayfalamakla neredeyse aynıdır:
```php
use App\Models\User;

$users = User::paginate(15);
```
Elbette, `where` cümleleri gibi sorguya başka kısıtlamalar ekledikten sonra da `paginate` metodunu çağırabilirsiniz:
```php
$users = User::where('votes', '>', 100)->paginate(15);
```
Eloquent modellerini sayfalarken `simplePaginate` metodunu da kullanabilirsiniz:
```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```
Benzer şekilde, Eloquent modellerini imleçli sayfalama (cursor pagination) ile sayfalamak için `cursorPaginate` metodunu kullanabilirsiniz:
```php
$users = User::where('votes', '>', 100)->cursorPaginate(15);
```

#### Sayfa Başına Birden Çok Sayfalayıcı Örneği (Multiple Paginator Instances per Page)
Bazen uygulamanız tarafından oluşturulan tek bir ekranda iki ayrı sayfalayıcı oluşturmanız gerekebilir. Ancak, her iki sayfalayıcı örneği de geçerli sayfayı saklamak için `page` sorgu dizesi parametresini kullanırsa, iki sayfalayıcı çakışacaktır. Bu çakışmayı çözmek için, `paginate`, `simplePaginate` ve `cursorPaginate` metotlarına sağlanan üçüncü argüman aracılığıyla, sayfalayıcının geçerli sayfasını saklamak için kullanmak istediğiniz sorgu dizesi parametresinin adını iletebilirsiniz:
```php
use App\Models\User;

$users = User::where('votes', '>', 100)->paginate(
    $perPage = 15, $columns = ['*'], $pageName = 'users'
);
```

### İmleçli Sayfalama (Cursor Pagination)
`paginate` ve `simplePaginate` SQL "offset" cümlesini kullanarak sorgular oluştururken, imleçli sayfalama (cursor pagination), sorguda bulunan sıralanmış sütunların değerlerini karşılaştıran "where" cümleleri oluşturarak çalışır ve Laravel'in tüm sayfalama metotları arasında mevcut en verimli veritabanı performansını sağlar. Bu sayfalama yöntemi, özellikle büyük veri kümeleri ve "sonsuz" kaydırma (infinite scrolling) kullanıcı arayüzleri için çok uygundur.

Sayfalayıcı tarafından oluşturulan URL'lerin sorgu dizesinde bir sayfa numarası içeren ofset tabanlı sayfalamanın aksine, imleç tabanlı sayfalama, sorgu dizesine bir "imleç" (cursor) dizesi yerleştirir. İmleç, bir sonraki sayfalanmış sorgunun sayfalamaya nereden başlaması gerektiğini ve hangi yönde sayfalaması gerektiğini içeren kodlanmış bir dizedir:
```
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```
Sorgu oluşturucu tarafından sunulan `cursorPaginate` metodu aracılığıyla imleç tabanlı bir sayfalayıcı örneği oluşturabilirsiniz. Bu metod, bir `Illuminate\Pagination\CursorPaginator` örneği döndürür:
```php
$users = DB::table('users')->orderBy('id')->cursorPaginate(15);
```
Bir imleç sayfalayıcı örneği aldıktan sonra, `paginate` ve `simplePaginate` metotlarını kullanırken tipik olarak yaptığınız gibi sayfalama sonuçlarını görüntüleyebilirsiniz. İmleç sayfalayıcı tarafından sunulan örnek metotları hakkında daha fazla bilgi için lütfen imleç sayfalayıcı örnek metodu dokümantasyonuna bakın.

Sorgunuzun, imleçli sayfalamadan yararlanmak için bir "order by" cümlesi içermesi gerekir. Ayrıca, sorgunun sıralandığı sütunlar, sayfaladığınız tabloya ait olmalıdır.

#### İmleçli vs. Ofsetli Sayfalama (Cursor vs. Offset Pagination)
Ofsetli sayfalama ve imleçli sayfalama arasındaki farkları göstermek için bazı örnek SQL sorgularını inceleyelim. Aşağıdaki sorguların her ikisi de `id`'ye göre sıralanmış bir `users` tablosu için "ikinci sayfa" sonuçlarını gösterecektir:
```sql
# Offset Pagination...
select * from users order by id asc limit 15 offset 15;

# Cursor Pagination...
select * from users where id > 15 order by id asc limit 15;
```
İmleçli sayfalama sorgusu, ofsetli sayfalamaya göre aşağıdaki avantajları sunar:

*   Büyük veri kümeleri için, "order by" sütunları indekslenmişse, imleçli sayfalama daha iyi performans sunar. Çünkü "offset" cümlesi önceden eşleşen tüm verileri tarar.
*   Sık yazma işlemleri olan veri kümeleri için, kullanıcının o anda görüntülemekte olduğu sayfaya yakın zamanda sonuçlar eklendiyse veya silindiyse, ofsetli sayfalama kayıtları atlayabilir veya kopyalar gösterebilir.

Ancak, imleçli sayfalamanın aşağıdaki sınırlamaları vardır:

*   `simplePaginate` gibi, imleçli sayfalama da yalnızca "Sonraki" ve "Önceki" bağlantılarını görüntülemek için kullanılabilir ve sayfa numaralarıyla bağlantılar oluşturmayı desteklemez.
*   Sıralamanın en az bir benzersiz sütuna veya benzersiz olan bir sütun kombinasyonuna dayanmasını gerektirir. `null` değerlere sahip sütunlar desteklenmez.
*   "Order by" cümlelerindeki sorgu ifadeleri (query expressions), yalnızca takma adlandırılıp (aliased) "select" cümlesine de eklenirlerse desteklenir.
*   Parametreli sorgu ifadeleri desteklenmez.

### Sayfalayıcıyı Manuel Olarak Oluşturma (Manually Creating a Paginator)
Bazen, halihazırda bellekte bulunan bir öğe dizisini (array of items) ileterek manuel olarak bir sayfalama örneği oluşturmak isteyebilirsiniz. Bunu, ihtiyaçlarınıza bağlı olarak bir `Illuminate\Pagination\Paginator`, `Illuminate\Pagination\LengthAwarePaginator` veya `Illuminate\Pagination\CursorPaginator` örneği oluşturarak yapabilirsiniz.

`Paginator` ve `CursorPaginator` sınıflarının sonuç kümesindeki toplam öğe sayısını bilmesi gerekmez; ancak, bu nedenle bu sınıfların son sayfanın indeksini almak için metotları yoktur. `LengthAwarePaginator`, `Paginator` ile neredeyse aynı argümanları kabul eder; ancak, sonuç kümesindeki toplam öğe sayısının bir sayısını gerektirir.

Başka bir deyişle, `Paginator`, sorgu oluşturucudaki `simplePaginate` metoduna, `CursorPaginator`, `cursorPaginate` metoduna ve `LengthAwarePaginator`, `paginate` metoduna karşılık gelir.

Manuel olarak bir sayfalayıcı örneği oluştururken, sayfalayıcıya ilettiğiniz sonuçlar dizisini manuel olarak "dilimlemelisiniz" (slice). Bunu nasıl yapacağınızdan emin değilseniz, PHP'nin `array_slice` fonksiyonuna göz atın.

### Sayfalama URL'lerini Özelleştirme (Customizing Pagination URLs)
Varsayılan olarak, sayfalayıcı tarafından oluşturulan bağlantılar, geçerli isteğin URI'siyle eşleşecektir. Ancak, sayfalayıcının `withPath` metodu, sayfalayıcının bağlantılar oluştururken kullandığı URI'yi özelleştirmenize olanak tanır. Örneğin, sayfalayıcının `http://example.com/admin/users?page=N` gibi bağlantılar oluşturmasını istiyorsanız, `withPath` metoduna `/admin/users` iletmelisiniz:
```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->withPath('/admin/users');

    // ...
});
```

#### Sorgu Dizesi Değerlerini Ekleme (Appending Query String Values)
`appends` metodunu kullanarak sayfalama bağlantılarının sorgu dizesine ekleme yapabilirsiniz. Örneğin, her sayfalama bağlantısına `sort=votes` eklemek için, `appends`'e aşağıdaki çağrıyı yapmalısınız:
```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->appends(['sort' => 'votes']);

    // ...
});
```
Geçerli isteğin tüm sorgu dizesi değerlerini sayfalama bağlantılarına eklemek isterseniz, `withQueryString` metodunu kullanabilirsiniz:
```php
$users = User::paginate(15)->withQueryString();
```

#### Hash Parçaları (Fragments) Ekleme
Sayfalayıcı tarafından oluşturulan URL'lere bir "hash parçası" (hash fragment) eklemeniz gerekiyorsa, `fragment` metodunu kullanabilirsiniz. Örneğin, her sayfalama bağlantısının sonuna `#users` eklemek için, `fragment` metodunu şu şekilde çağırmalısınız:
```php
$users = User::paginate(15)->fragment('users');
```

## Sayfalama Sonuçlarını Görüntüleme (Displaying Pagination Results)

`paginate` metodunu çağırdığınızda, bir `Illuminate\Pagination\LengthAwarePaginator` örneği alırsınız; `simplePaginate` metodunu çağırmak ise bir `Illuminate\Pagination\Paginator` örneği döndürür. Ve son olarak, `cursorPaginate` metodunu çağırmak bir `Illuminate\Pagination\CursorPaginator` örneği döndürür.

Bu nesneler, sonuç kümesini tanımlayan birkaç metot sağlar. Bu yardımcı metotlara ek olarak, sayfalayıcı örnekleri yineleyicilerdir (iterators) ve bir dizi olarak döngüye alınabilirler. Yani, sonuçları aldıktan sonra, sonuçları görüntüleyebilir ve sayfa bağlantılarını Blade kullanarak oluşturabilirsiniz:
```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```
`links` metodu, sonuç kümesindeki diğer sayfalara olan bağlantıları oluşturacaktır. Bu bağlantıların her biri zaten uygun `page` sorgu dizesi değişkenini içerecektir. Unutmayın, `links` metodu tarafından oluşturulan HTML, Tailwind CSS framework'ü ile uyumludur.

### Sayfalama Bağlantı Penceresini Ayarlama (Adjusting the Pagination Link Window)
Sayfalayıcı sayfalama bağlantılarını görüntülerken, geçerli sayfa numarasının yanı sıra geçerli sayfadan önceki ve sonraki üç sayfanın bağlantıları da görüntülenir. `onEachSide` metodunu kullanarak, sayfalayıcı tarafından oluşturulan bağlantıların orta, kayan penceresinde (sliding window) geçerli sayfanın her iki tarafında görüntülenen ek bağlantı sayısını kontrol edebilirsiniz:
```blade
{{ $users->onEachSide(5)->links() }}
```

### Sonuçları JSON'a Dönüştürme (Converting Results to JSON)
Laravel sayfalayıcı sınıfları, `Illuminate\Contracts\Support\Jsonable` sözleşmesini (contract) uygular ve `toJson` metodunu kullanıma sunar, bu nedenle sayfalama sonuçlarınızı JSON'a dönüştürmek çok kolaydır. Ayrıca, bir sayfalayıcı örneğini bir rotadan veya controller eyleminden döndürerek JSON'a dönüştürebilirsiniz:
```php
use App\Models\User;

Route::get('/users', function () {
    return User::paginate();
});
```
Sayfalayıcıdan gelen JSON, `total`, `current_page`, `last_page` ve daha fazlası gibi meta bilgileri içerecektir. Sonuç kayıtlarına JSON dizisindeki `data` anahtarı aracılığıyla erişilebilir. İşte bir rotadan bir sayfalayıcı örneği döndürülerek oluşturulan JSON'un bir örneği:
```json
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "current_page_url": "http://laravel.app?page=1",
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Kayıt...
        },
        {
            // Kayıt...
        }
   ]
}
```

## Sayfalama Görünümünü Özelleştirme (Customizing the Pagination View)

Varsayılan olarak, sayfalama bağlantılarını görüntülemek için oluşturulan görünümler, Tailwind CSS framework'ü ile uyumludur. Ancak, Tailwind kullanmıyorsanız, bu bağlantıları oluşturmak için kendi görünümlerinizi tanımlamakta özgürsünüz. Bir sayfalayıcı örneğinde `links` metodunu çağırırken, metoda ilk argüman olarak görünüm adını iletebilirsiniz:
```blade
{{ $paginator->links('view.name') }}

<!-- Görünüme ek veri aktarma... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```
Ancak, sayfalama görünümlerini özelleştirmenin en kolay yolu, `vendor:publish` komutunu kullanarak bunları `resources/views/vendor` dizininize aktarmaktır (export):
```bash
php artisan vendor:publish --tag=laravel-pagination
```
Bu komut, görünümleri uygulamanızın `resources/views/vendor/pagination` dizinine yerleştirecektir. Bu dizindeki `tailwind.blade.php` dosyası, varsayılan sayfalama görünümüne karşılık gelir. Sayfalama HTML'ini değiştirmek için bu dosyayı düzenleyebilirsiniz.

Farklı bir dosyayı varsayılan sayfalama görünümü olarak belirlemek isterseniz, `App\Providers\AppServiceProvider` sınıfınızın `boot` metodu içinde sayfalayıcının `defaultView` ve `defaultSimpleView` metotlarını çağırabilirsiniz:
```php
<?php

namespace App\Providers;

use Illuminate\Pagination\Paginator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }
}
```

### Bootstrap Kullanma (Using Bootstrap)
Laravel, Bootstrap CSS kullanılarak oluşturulmuş sayfalama görünümleri içerir. Varsayılan Tailwind görünümleri yerine bu görünümleri kullanmak için, sayfalayıcının `useBootstrapFour` veya `useBootstrapFive` metotlarını çağırabilirsiniz. Tipik olarak, bu metodu uygulamanızın `AppServiceProvider`'ının `boot` metodunda çağırmalısınız:
```php
use Illuminate\Pagination\Paginator;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```