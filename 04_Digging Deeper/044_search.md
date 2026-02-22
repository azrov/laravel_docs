# Laravel 12 Dokümantasyonu: Arama (Search)

## Giriş

Neredeyse her uygulamanın arama özelliğine ihtiyacı vardır. Kullanıcılarınız bir bilgi tabanında ilgili makaleleri arıyor, bir ürün kataloğunu keşfediyor veya bir belge kümesine karşı doğal dilde sorular soruyor olsun, Laravel bu senaryoların her birini ele almak için yerleşik araçlar sağlar - ve çoğu zaman bu noktaya ulaşmak için herhangi bir harici servise ihtiyacınız olmaz.

Çoğu uygulama, Laravel tarafından sağlanan yerleşik veritabanı destekli seçeneklerin fazlasıyla yeterli olduğunu görecektir - harici arama servisleri yalnızca yazım hatası toleransı (typo tolerance), yönlü filtreleme (faceted filtering) veya büyük ölçekte coğrafi arama (geo-search) gibi özelliklere ihtiyacınız olduğunda gereklidir.

#### Tam Metin Arama (Full-Text Search)
Anahtar kelime alaka sıralamasına (keyword relevance ranking) ihtiyacınız olduğunda - veritabanının, sonuçları arama terimleriyle ne kadar iyi eşleştiklerine göre puanladığı ve sıraladığı durum - Laravel'in `whereFullText` sorgu oluşturucu (query builder) metodu, MariaDB, MySQL ve PostgreSQL'deki yerel tam metin indekslerinden (native full-text indexes) yararlanır. Tam metin arama, kelime sınırlarını ve kök bulmayı (stemming) anlar, böylece "running" için yapılan bir arama, "run" içeren kayıtlarla eşleşebilir. Hiçbir harici servis gerekmez.

#### Anlamsal / Vektör Arama (Semantic / Vector Search)
Kesin anahtar kelimeler yerine anlamla eşleşen sonuçları bulan, yapay zeka destekli anlamsal arama için `whereVectorSimilarTo` sorgu oluşturucu metodu, `pgvector` eklentisiyle PostgreSQL'de saklanan vektör yerleştirmelerini (vector embeddings) kullanır. Örneğin, "best wineries in Napa Valley" (Napa Vadisi'ndeki en iyi şarap imalathaneleri) araması, kelimeler örtüşmese bile "Top Vineyards to Visit" (Ziyaret Edilecek En İyi Bağlar) başlıklı bir makaleyi yüzeye çıkarabilir. Vektör arama, `pgvector` eklentisine sahip PostgreSQL ve Laravel AI SDK gerektirir.

#### Yeniden Sıralama (Reranking)
Laravel'in AI SDK'sı, herhangi bir sonuç kümesini bir sorguyla anlamsal alaka düzeyine göre yeniden sıralamak için yapay zeka modellerini kullanan yeniden sıralama yetenekleri sağlar. Yeniden sıralama, özellikle tam metin arama gibi hızlı bir ilk alma adımından (initial retrieval step) sonra ikinci bir aşama olarak son derece güçlüdür - size hem hız hem de anlamsal doğruluk kazandırır.

#### Laravel Scout Arama
Arama indekslerini Eloquent modelleriyle otomatik olarak senkronize tutan bir `Searchable` özelliği (trait) isteyen uygulamalar için Laravel Scout, hem yerleşik bir veritabanı motoru (database engine) hem de Algolia, Meilisearch ve Typesense gibi üçüncü taraf servisler için sürücüler (drivers) sunar.

## Tam Metin Arama (Full-Text Search)

`LIKE` sorguları basit alt dize eşleştirme (substring matching) için iyi çalışırken, dili anlamazlar. "running" için yapılan bir `LIKE` araması, "run" içeren bir kaydı bulamaz ve sonuçlar alaka düzeyine göre sıralanmaz - bunlar basitçe veritabanının onları bulduğu sırayla döndürülür. Tam metin arama, kelime sınırlarını, kök bulmayı ve alaka puanlamasını (relevance scoring) anlayan özel indeksler kullanarak bu sorunların her ikisini de çözer ve veritabanının en alakalı sonuçları ilk sırada döndürmesine olanak tanır.

Hızlı tam metin arama, MariaDB, MySQL ve PostgreSQL'e entegre edilmiştir - hiçbir harici arama servisi gerekmez. Yalnızca aramak istediğiniz sütunlara bir tam metin indeksi eklemeniz ve ardından bunlara karşı arama yapmak için `whereFullText` sorgu oluşturucu metodunu kullanmanız yeterlidir.

Tam metin arama şu anda MariaDB, MySQL ve PostgreSQL tarafından desteklenmektedir.

### Tam Metin İndeksleri Ekleme (Adding Full-Text Indexes)
Tam metin aramayı kullanmak için önce aramak istediğiniz sütunlara bir tam metin indeksi ekleyin. İndeksi tek bir sütuna ekleyebilir veya birden çok alanda aynı anda arama yapan bir bileşik indeks (composite index) oluşturmak için bir sütun dizisi iletebilirsiniz:
```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('body');
    $table->timestamps();

    $table->fullText(['title', 'body']);
});
```
PostgreSQL'de, indeks için bir dil yapılandırması (language configuration) belirtebilirsiniz; bu, kelimelerin nasıl köklenerek alınacağını (stemmed) kontrol eder:
```php
$table->fullText('body')->language('english');
```
İndeks oluşturma hakkında daha fazla bilgi için migration dokümantasyonuna bakın.

### Tam Metin Sorguları Çalıştırma (Running Full-Text Queries)
İndeks yerleştirildikten sonra, ona karşı arama yapmak için `whereFullText` sorgu oluşturucu metodunu kullanın. Laravel, veritabanı sürücünüz için uygun SQL'i oluşturacaktır - örneğin, MariaDB ve MySQL'de `MATCH(...) AGAINST(...)`, PostgreSQL'de ise `to_tsvector(...) @@ plainto_tsquery(...)`:
```php
$articles = Article::whereFullText('body', 'web developer')->get();
```
MariaDB ve MySQL kullanırken, sonuçlar otomatik olarak alaka puanına (relevance score) göre sıralanır. PostgreSQL'de `whereFullText` eşleşen kayıtları filtreler ancak bunları alaka düzeyine göre sıralamaz - PostgreSQL'de otomatik alaka sıralamasına ihtiyacınız varsa, bunu sizin için halleden Scout'un veritabanı motorunu (database engine) kullanmayı düşünün.

Birden çok sütun arasında bileşik bir tam metin indeksi oluşturduysanız, `whereFullText`'e aynı sütun dizisini ileterek tümüne karşı arama yapabilirsiniz:
```php
$articles = Article::whereFullText(
    ['title', 'body'], 'web developer'
)->get();
```
`orWhereFullText` metodu, bir "veya" (or) koşulu olarak bir tam metin arama cümlesi eklemek için kullanılabilir. Tam ayrıntılar için sorgu oluşturucu dokümantasyonuna bakın.

## Anlamsal / Vektör Arama (Semantic / Vector Search)

Tam metin arama, anahtar kelimelerin eşleştirilmesine dayanır - sorgudaki kelimelerin verilerde (bir biçimde) görünmesi gerekir. Anlamsal arama (semantic search) temelde farklı bir yaklaşım benimser: metnin anlamını sayı dizileri olarak temsil etmek için yapay zeka tarafından oluşturulmuş vektör yerleştirmelerini (vector embeddings) kullanır ve ardından anlamı sorguya en çok benzeyen sonuçları bulur. Örneğin, "best wineries in Napa Valley" (Napa Vadisi'ndeki en iyi şarap imalathaneleri) araması, kelimeler hiç örtüşmese bile "Top Vineyards to Visit" (Ziyaret Edilecek En İyi Bağlar) başlıklı bir makaleyi yüzeye çıkarabilir.

Vektör araması için temel iş akışı şudur: her bir içerik parçası için bir yerleştirme (sayısal bir dizi) oluşturun ve bunu verilerinizin yanında saklayın, ardından arama zamanında kullanıcının sorgusu için bir yerleştirme oluşturun ve vektör uzayında ona en yakın olan saklanmış yerleştirmeleri bulun.

Vektör araması, `pgvector` eklentisine sahip bir PostgreSQL veritabanı ve Laravel AI SDK gerektirir. Tüm Laravel Cloud Serverless Postgres veritabanları zaten `pgvector` içerir.

### Yerleştirmeler (Embeddings) Oluşturma
Bir yerleştirme (embedding), bir metin parçasının anlamsal anlamını temsil eden yüksek boyutlu bir sayısal dizidir (tipik olarak yüzlerce veya binlerce sayı). Laravel'in `Stringable` sınıfında bulunan `toEmbeddings` metodunu kullanarak bir dize için yerleştirmeler oluşturabilirsiniz:
```php
use Illuminate\Support\Str;

$embedding = Str::of('Napa Valley has great wine.')->toEmbeddings();
```
Aynı anda birden çok girdi için yerleştirmeler oluşturmak için - bu, yerleştirme sağlayıcısına (embedding provider) yalnızca tek bir API çağrısı gerektirdiğinden, bunları tek tek oluşturmaktan daha verimlidir - `Embeddings` sınıfını kullanın:
```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```
Yerleştirme sağlayıcılarını yapılandırma, boyutları özelleştirme ve önbelleğe alma hakkında daha fazla ayrıntı için AI SDK dokümantasyonuna bakın.

### Vektörleri Saklama ve İndeksleme (Storing and Indexing Vectors)
Vektör yerleştirmelerini saklamak için migration'ınızda bir `vector` sütunu tanımlayın ve yerleştirme sağlayıcınızın çıktısıyla eşleşen boyut sayısını belirtin (örneğin, OpenAI'nin `text-embedding-3-small` modeli için 1536). Ayrıca, büyük veri kümelerinde benzerlik aramalarını önemli ölçüde hızlandıran bir HNSW (Hierarchical Navigable Small World) indeksi oluşturmak için sütunda `index` metodunu çağırmalısınız:
```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536)->index();
    $table->timestamps();
});
```
`Schema::ensureVectorExtensionExists` metodu, tabloyu oluşturmadan önce PostgreSQL veritabanınızda `pgvector` eklentisinin etkinleştirildiğinden emin olur.

Eloquent modelinizde, vektör sütununu bir `array` olarak dönüştürün (cast), böylece Laravel, PHP dizileri ile veritabanının vektör formatı arasındaki dönüşümü otomatik olarak halleder:
```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```
Vektör sütunları ve indeksleri hakkında daha fazla ayrıntı için migration dokümantasyonuna bakın.

### Benzerliğe Göre Sorgulama (Querying by Similarity)
İçeriğiniz için yerleştirmeleri sakladıktan sonra, `whereVectorSimilarTo` metodunu kullanarak benzer kayıtları arayabilirsiniz. Bu metod, verilen yerleştirmeyi saklanmış vektörlerle kosinüs benzerliği (cosine similarity) kullanarak karşılaştırır, `minSimilarity` eşiğinin altındaki sonuçları filtreler ve sonuçları otomatik olarak alaka düzeyine göre sıralar - en benzer kayıtlar ilk sırada olacak şekilde. Eşik, 0.0 ile 1.0 arasında bir değer olmalıdır, burada 1.0 vektörlerin aynı olduğu anlamına gelir:
```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```
Kolaylık olması açısından, bir yerleştirme dizisi yerine düz bir dize verildiğinde, Laravel, yapılandırılmış yerleştirme sağlayıcınızı kullanarak sizin için otomatik olarak yerleştirmeyi oluşturacaktır. Bu, kullanıcının arama sorgusunu önce manuel olarak bir yerleştirmeye dönüştürmeden doğrudan iletebileceğiniz anlamına gelir:
```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```
Vektör sorguları üzerinde daha düşük seviyeli kontrol için `whereVectorDistanceLessThan`, `selectVectorDistance` ve `orderByVectorDistance` metotları da mevcuttur. Bu metotlar, benzerlik puanları yerine doğrudan mesafe değerleriyle çalışmanıza, hesaplanan mesafeyi sonuçlarınızda bir sütun olarak seçmenize veya sıralamayı manuel olarak kontrol etmenize olanak tanır. Tam ayrıntılar için sorgu oluşturucu dokümantasyonuna ve AI SDK dokümantasyonuna bakın.

## Sonuçları Yeniden Sıralama (Reranking Results)

Yeniden sıralama (reranking), bir yapay zeka modelinin bir sonuç kümesini, her bir sonucun belirli bir sorguyla ne kadar anlamsal olarak alakalı olduğuna göre yeniden sıraladığı bir tekniktir. Yerleştirmeleri önceden hesaplamanızı ve saklamanızı gerektiren vektör aramasının aksine, yeniden sıralama herhangi bir metin koleksiyonu üzerinde çalışır - girdi olarak ham içeriği ve sorguyu alır ve öğeleri alaka düzeyine göre sıralanmış olarak döndürür.

Yeniden sıralama, özellikle hızlı bir ilk alma adımından sonra ikinci bir aşama olarak son derece güçlüdür. Örneğin, binlerce kaydı hızla ilk 50 adaya indirmek için tam metin aramayı kullanabilir ve ardından en alakalı sonuçları en üste koymak için yeniden sıralamayı kullanabilirsiniz. Bu "al ve yeniden sırala" (retrieve then rerank) deseni, size hem hız hem de anlamsal doğruluk kazandırır.

`Reranking` sınıfını kullanarak bir dize dizisini yeniden sıralayabilirsiniz:
```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

$response->first()->document; // "Laravel is a PHP web application framework."
```
Laravel koleksiyonları ayrıca bir alan adı (veya closure) ve bir sorgu kabul eden bir `rerank` makrosuna sahiptir, bu da Eloquent sonuçlarını yeniden sıralamayı kolaylaştırır:
```php
$articles = Article::all()
    ->rerank('body', 'Laravel tutorials');
```
Yeniden sıralama sağlayıcılarını yapılandırma ve mevcut seçenekler hakkında tam ayrıntılar için AI SDK dokümantasyonuna bakın.

## Laravel Scout

Yukarıda açıklanan arama tekniklerinin tümü, doğrudan kodunuzda çağırdığınız sorgu oluşturucu metotlarıdır. Laravel Scout farklı bir yaklaşım benimser: Eloquent modellerinize eklediğiniz bir `Searchable` özelliği (trait) sağlar ve Scout, kayıtlar oluşturuldukça, güncellendikçe ve silindikçe arama indekslerinizi otomatik olarak senkronize tutar. Bu, indeks güncellemelerini manuel olarak yönetmek zorunda kalmadan modellerinizin her zaman aranabilir olmasını istediğinizde özellikle kullanışlıdır.

### Veritabanı Motoru (Database Engine)
Scout'un yerleşik veritabanı motoru, mevcut veritabanınıza karşı tam metin ve `LIKE` aramaları gerçekleştirir - hiçbir harici servis veya ek altyapı gerekmez. Basitçe modelinize `Searchable` özelliğini ekleyin ve aranabilir olmasını istediğiniz sütunları döndüren bir `toSearchableArray` metodu tanımlayın.

Her sütun için arama stratejisini kontrol etmek üzere PHP niteliklerini (attributes) kullanabilirsiniz. `SearchUsingFullText`, veritabanınızın tam metin indeksini kullanacaktır, `SearchUsingPrefix` yalnızca dizenin başından itibaren eşleştirecektir (`example%`) ve herhangi bir nitelik içermeyen sütunlar, her iki tarafta joker karakterlerle (`%example%`) varsayılan bir `LIKE` stratejisi kullanır:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;
use Laravel\Scout\Searchable;

class Article extends Model
{
    use Searchable;

    #[SearchUsingPrefix(['id'])]
    #[SearchUsingFullText(['title', 'body'])]
    public function toSearchableArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
        ];
    }
}
```
Bir sütunun tam metin sorgu kısıtlamaları kullanması gerektiğini belirtmeden önce, sütuna bir tam metin indeksi atandığından emin olun.

Özellik eklendikten sonra, modelinizi Scout'un `search` metodunu kullanarak arayabilirsiniz. Scout'un veritabanı motoru, sonuçları otomatik olarak alaka düzeyine göre sıralayacaktır, hatta PostgreSQL'de bile:
```php
$articles = Article::search('Laravel')->get();
```
Veritabanı motoru, arama ihtiyaçlarınız orta düzeyde olduğunda ve harici bir servis dağıtmadan Scout'un otomatik indeks senkronizasyonunun rahatlığını istediğinizde harika bir seçimdir. Filtreleme, sayfalandırma (pagination) ve yazılı olarak silinmiş (soft-deleted) kayıtların işlenmesi dahil olmak üzere en yaygın arama kullanım durumlarını iyi bir şekilde ele alır. Tam ayrıntılar için Scout dokümantasyonuna bakın.

### Üçüncü Taraf Motorlar (Third-Party Engines)
Scout ayrıca Algolia, Meilisearch ve Typesense gibi üçüncü taraf arama motorlarını da destekler. Bu özel arama servisleri, yazım hatası toleransı, yönlü filtreleme, coğrafi arama ve özel sıralama kuralları gibi gelişmiş özellikler sunar - bu özellikler, çok büyük ölçekte veya yazarken arama (search-as-you-type) deneyimine ihtiyacınız olduğunda önemli hale gelir.

Scout, tüm sürücüleri arasında birleşik bir API sağladığından, daha sonra veritabanı motorundan üçüncü taraf bir motora geçmek minimum kod değişikliği gerektirir. Veritabanı motoruyla başlayabilir ve yalnızca uygulamanızın ihtiyaçları veritabanının sağlayabileceklerini aşarsa üçüncü taraf bir servise geçiş yapabilirsiniz.

Üçüncü taraf motorları yapılandırma hakkında tam ayrıntılar için Scout dokümantasyonuna bakın.

Birçok uygulama asla harici bir arama motoruna ihtiyaç duymaz. Bu sayfada açıklanan yerleşik teknikler, kullanım durumlarının büyük çoğunluğunu kapsar.