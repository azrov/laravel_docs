# Laravel 12 Dokümantasyonu: Dizeler (Strings)

## Giriş

Laravel, dize değerlerini (string values) işlemek için çeşitli fonksiyonlar içerir. Bu fonksiyonların çoğu framework'ün kendisi tarafından kullanılır; ancak, kullanışlı bulursanız bunları kendi uygulamalarınızda kullanmakta özgürsünüz.

## Mevcut Metotlar (Available Methods)

### Dizeler (Strings)
Bu bölüm, dize manipülasyonu için kullanılan çeşitli global yardımcı fonksiyonları ve `Str` sınıfı metotlarını içerir.

#### `__()` Fonksiyonu
`__` fonksiyonu, verilen çeviri dizesini veya çeviri anahtarını dil dosyalarınızı kullanarak çevirir:
```php
echo __('Uygulamamıza hoş geldiniz');

echo __('messages.welcome');
```
Belirtilen çeviri dizesi veya anahtarı mevcut değilse, `__` fonksiyonu verilen değeri döndürecektir. Yani, yukarıdaki örneği kullanırsak, bu çeviri anahtarı mevcut değilse `__` fonksiyonu `messages.welcome` döndürecektir.

#### `class_basename()` Fonksiyonu
`class_basename` fonksiyonu, verilen sınıfın adını, sınıfın ad alanı (namespace) kaldırılmış olarak döndürür:
```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```

#### `e()` Fonksiyonu
`e` fonksiyonu, PHP'nin `htmlspecialchars` fonksiyonunu varsayılan olarak `double_encode` seçeneği `true` olacak şekilde çalıştırır:
```php
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

#### `preg_replace_array()` Fonksiyonu
`preg_replace_array` fonksiyonu, bir diziyi kullanarak bir dizedeki belirli bir kalıbı (pattern) sırayla değiştirir:
```php
$string = 'Etkinlik :start ile :end arasında gerçekleşecek';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// Etkinlik 8:30 ile 9:00 arasında gerçekleşecek
```

#### `Str::after()` Metodu
`Str::after` metodu, bir dizede verilen değerden sonraki her şeyi döndürür. Değer dize içinde mevcut değilse, dizenin tamamı döndürülecektir:
```php
use Illuminate\Support\Str;

$slice = Str::after('Bu benim adım', 'Bu');

// ' benim adım'
```

#### `Str::afterLast()` Metodu
`Str::afterLast` metodu, bir dizede verilen değerin son geçtiği yerden sonraki her şeyi döndürür. Değer dize içinde mevcut değilse, dizenin tamamı döndürülecektir:
```php
use Illuminate\Support\Str;

$slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

// 'Controller'
```

#### `Str::apa()` Metodu
`Str::apa` metodu, verilen dizeyi APA yönergelerini izleyerek başlık büyük harfli (title case) biçime dönüştürür:
```php
use Illuminate\Support\Str;

$title = Str::apa('Creating A Project');

// 'Creating a Project'
```

#### `Str::ascii()` Metodu
`Str::ascii` metodu, dizeyi bir ASCII değerine dönüştürmeye (transliterate) çalışacaktır:
```php
use Illuminate\Support\Str;

$slice = Str::ascii('û');

// 'u'
```

#### `Str::before()` Metodu
`Str::before` metodu, bir dizede verilen değerden önceki her şeyi döndürür:
```php
use Illuminate\Support\Str;

$slice = Str::before('Bu benim adım', 'adım');

// 'Bu benim '
```

#### `Str::beforeLast()` Metodu
`Str::beforeLast` metodu, bir dizede verilen değerin son geçtiği yerden önceki her şeyi döndürür:
```php
use Illuminate\Support\Str;

$slice = Str::beforeLast('Bu benim adım', 'im');

// 'Bu ben'
```

#### `Str::between()` Metodu
`Str::between` metodu, bir dizenin iki değer arasındaki kısmını döndürür:
```php
use Illuminate\Support\Str;

$slice = Str::between('Bu benim adım', 'Bu', 'adım');

// ' benim '
```

#### `Str::betweenFirst()` Metodu
`Str::betweenFirst` metodu, bir dizenin iki değer arasındaki mümkün olan en küçük kısmını döndürür:
```php
use Illuminate\Support\Str;

$slice = Str::betweenFirst('[a] bc [d]', '[', ']');

// 'a'
```

#### `Str::camel()` Metodu
`Str::camel` metodu, verilen dizeyi camelCase'e dönüştürür:
```php
use Illuminate\Support\Str;

$converted = Str::camel('foo_bar');

// 'fooBar'
```

#### `Str::charAt()` Metodu
`Str::charAt` metodu, belirtilen indisteki (index) karakteri döndürür. İndeks sınırların dışındaysa, `false` döndürülür:
```php
use Illuminate\Support\Str;

$character = Str::charAt('Bu benim adım.', 6);

// 'i'
```

#### `Str::chopStart()` Metodu
`Str::chopStart` metodu, verilen değerin ilk geçtiği yeri, yalnızca değer dizenin başında görünüyorsa kaldırır:
```php
use Illuminate\Support\Str;

$url = Str::chopStart('https://laravel.com', 'https://');

// 'laravel.com'
```
İkinci argüman olarak bir dizi de iletebilirsiniz. Dize, dizideki değerlerden herhangi biriyle başlıyorsa, bu değer dizeden kaldırılacaktır:
```php
use Illuminate\Support\Str;

$url = Str::chopStart('http://laravel.com', ['https://', 'http://']);

// 'laravel.com'
```

#### `Str::chopEnd()` Metodu
`Str::chopEnd` metodu, verilen değerin son geçtiği yeri, yalnızca değer dizenin sonunda görünüyorsa kaldırır:
```php
use Illuminate\Support\Str;

$url = Str::chopEnd('app/Models/Photograph.php', '.php');

// 'app/Models/Photograph'
```
İkinci argüman olarak bir dizi de iletebilirsiniz. Dize, dizideki değerlerden herhangi biriyle bitiyorsa, bu değer dizeden kaldırılacaktır:
```php
use Illuminate\Support\Str;

$url = Str::chopEnd('laravel.com/index.php', ['/index.html', '/index.php']);

// 'laravel.com'
```

#### `Str::contains()` Metodu
`Str::contains` metodu, verilen dizenin verilen değeri içerip içermediğini belirler. Varsayılan olarak, bu metod büyük/küçük harf duyarlıdır (case sensitive):
```php
use Illuminate\Support\Str;

$contains = Str::contains('Bu benim adım', 'benim');

// true
```
Ayrıca, verilen dizenin dizideki değerlerden herhangi birini içerip içermediğini belirlemek için bir değerler dizisi iletebilirsiniz:
```php
use Illuminate\Support\Str;

$contains = Str::contains('Bu benim adım', ['benim', 'foo']);

// true
```
`ignoreCase` argümanını `true` olarak ayarlayarak büyük/küçük harf duyarlılığını devre dışı bırakabilirsiniz:
```php
use Illuminate\Support\Str;

$contains = Str::contains('Bu benim adım', 'BENİM', ignoreCase: true);

// true
```

#### `Str::containsAll()` Metodu
`Str::containsAll` metodu, verilen dizenin belirli bir dizideki tüm değerleri içerip içermediğini belirler:
```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('Bu benim adım', ['benim', 'adım']);

// true
```
`ignoreCase` argümanını `true` olarak ayarlayarak büyük/küçük harf duyarlılığını devre dışı bırakabilirsiniz:
```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('Bu benim adım', ['BENİM', 'ADIM'], ignoreCase: true);

// true
```

#### `Str::doesntContain()` Metodu
`Str::doesntContain` metodu, verilen dizenin verilen değeri içermediğini belirler. Varsayılan olarak, bu metod büyük/küçük harf duyarlıdır:
```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('Bu adım', 'benim');

// true
```
Ayrıca, verilen dizenin dizideki değerlerden herhangi birini içermediğini belirlemek için bir değerler dizisi iletebilirsiniz:
```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('Bu adım', ['benim', 'framework']);

// true
```
`ignoreCase` argümanını `true` olarak ayarlayarak büyük/küçük harf duyarlılığını devre dışı bırakabilirsiniz:
```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('Bu adım', 'BENİM', ignoreCase: true);

// true
```

#### `Str::deduplicate()` Metodu
`Str::deduplicate` metodu, bir dizede bir karakterin ardışık örneklerini, o karakterin tek bir örneğiyle değiştirir. Varsayılan olarak, metod boşlukları (spaces) tekilleştirir (deduplicates):
```php
use Illuminate\Support\Str;

$result = Str::deduplicate('The   Laravel   Framework');

// The Laravel Framework
```
Metoda ikinci argüman olarak ileterek tekilleştirilecek farklı bir karakter belirtebilirsiniz:
```php
use Illuminate\Support\Str;

$result = Str::deduplicate('The---Laravel---Framework', '-');

// The-Laravel-Framework
```

#### `Str::doesntEndWith()` Metodu
`Str::doesntEndWith` metodu, verilen dizenin verilen değerle bitmediğini belirler:
```php
use Illuminate\Support\Str;

$result = Str::doesntEndWith('Bu benim adım', 'kedi');

// true
```
Ayrıca, verilen dizenin dizideki değerlerden herhangi biriyle bitmediğini belirlemek için bir değerler dizisi iletebilirsiniz:
```php
use Illuminate\Support\Str;

$result = Str::doesntEndWith('Bu benim adım', ['bu', 'foo']);

// true

$result = Str::doesntEndWith('Bu benim adım', ['adım', 'foo']);

// false
```

#### `Str::doesntStartWith()` Metodu
`Str::doesntStartWith` metodu, verilen dizenin verilen değerle başlamadığını belirler:
```php
use Illuminate\Support\Str;

$result = Str::doesntStartWith('Bu benim adım', 'Şu');

// true
```
Bir olası değerler dizisi iletilirse, `doesntStartWith` metodu, dize verilen değerlerden hiçbiriyle başlamıyorsa `true` döndürecektir:
```php
$result = Str::doesntStartWith('Bu benim adım', ['Ne', 'Şu', 'Orada']);

// true
```

#### `Str::endsWith()` Metodu
`Str::endsWith` metodu, verilen dizenin verilen değerle bitip bitmediğini belirler:
```php
use Illuminate\Support\Str;

$result = Str::endsWith('Bu benim adım', 'adım');

// true
```
Ayrıca, verilen dizenin dizideki değerlerden herhangi biriyle bitip bitmediğini belirlemek için bir değerler dizisi iletebilirsiniz:
```php
use Illuminate\Support\Str;

$result = Str::endsWith('Bu benim adım', ['adım', 'foo']);

// true

$result = Str::endsWith('Bu benim adım', ['bu', 'foo']);

// false
```

#### `Str::excerpt()` Metodu
`Str::excerpt` metodu, belirli bir dizeden, o dize içindeki bir ifadenin ilk örneğiyle eşleşen bir alıntı (excerpt) çıkarır:
```php
use Illuminate\Support\Str;

$excerpt = Str::excerpt('Bu benim adım', 'benim', [
    'radius' => 3
]);

// '... benim a...'
```
Varsayılan olarak `100` olan `radius` seçeneği, kısaltılmış dizenin her iki tarafında görünmesi gereken karakter sayısını tanımlamanıza olanak tanır.

Ek olarak, kısaltılmış dizeye ön ek (prepend) ve son ek (append) olarak eklenecek dizeyi tanımlamak için `omission` seçeneğini kullanabilirsiniz:
```php
use Illuminate\Support\Str;

$excerpt = Str::excerpt('Bu benim adım', 'adım', [
    'radius' => 3,
    'omission' => '(...) '
]);

// '(...) adım'
```

#### `Str::finish()` Metodu
`Str::finish` metodu, bir dize zaten o değerle bitmiyorsa, o dizeye verilen değerin tek bir örneğini ekler:
```php
use Illuminate\Support\Str;

$adjusted = Str::finish('bu/dize', '/');

// bu/dize/

$adjusted = Str::finish('bu/dize/', '/');

// bu/dize/
```

#### `Str::fromBase64()` Metodu
`Str::fromBase64` metodu, verilen Base64 dizesinin kodunu çözer (decodes):
```php
use Illuminate\Support\Str;

$decoded = Str::fromBase64('TGFyYXZlbA==');

// Laravel
```

#### `Str::headline()` Metodu
`Str::headline` metodu, büyük/küçük harf, kısa çizgiler veya alt çizgilerle ayrılmış dizeleri, her kelimenin ilk harfi büyük olacak şekilde boşlukla ayrılmış bir dizeye dönüştürecektir:
```php
use Illuminate\Support\Str;

$headline = Str::headline('steve_jobs');

// Steve Jobs

$headline = Str::headline('EmailNotificationSent');

// Email Notification Sent
```

#### `Str::inlineMarkdown()` Metodu
`Str::inlineMarkdown` metodu, GitHub tarzı Markdown'ı (GitHub flavored Markdown) CommonMark kullanarak satır içi HTML'ye (inline HTML) dönüştürür. Ancak, `markdown` metodunun aksine, oluşturulan tüm HTML'yi bir blok düzeyinde öğeye (block-level element) sarmaz (wrap):
```php
use Illuminate\Support\Str;

$html = Str::inlineMarkdown('**Laravel**');

// <strong>Laravel</strong>
```

#### Markdown Güvenliği (Markdown Security)
Varsayılan olarak Markdown, ham HTML'yi (raw HTML) destekler; bu, ham kullanıcı girdisiyle kullanıldığında Siteler Arası Betik Çalıştırma (Cross-Site Scripting - XSS) güvenlik açıklarına yol açabilir. CommonMark Güvenlik dokümantasyonuna göre, ham HTML'yi kaçış karakteri eklemek (escape) veya kaldırmak (strip) için `html_input` seçeneğini ve güvenli olmayan bağlantılara izin verilip verilmeyeceğini belirtmek için `allow_unsafe_links` seçeneğini kullanabilirsiniz. Biraz ham HTML'ye izin vermeniz gerekiyorsa, derlenmiş Markdown'ınızı bir HTML arındırıcıdan (HTML Purifier) geçirmelisiniz:
```php
use Illuminate\Support\Str;

Str::inlineMarkdown('Enjekte: <script>alert("Merhaba XSS!");</script>', [
    'html_input' => 'strip',
    'allow_unsafe_links' => false,
]);

// Enjekte: alert("Merhaba XSS!");
```

#### `Str::is()` Metodu
`Str::is` metodu, verilen bir dizenin belirli bir kalıpla (pattern) eşleşip eşleşmediğini belirler. Yıldız işaretleri (`*`) joker karakter olarak kullanılabilir:
```php
use Illuminate\Support\Str;

$matches = Str::is('foo*', 'foobar');

// true

$matches = Str::is('baz*', 'foobar');

// false
```
`ignoreCase` argümanını `true` olarak ayarlayarak büyük/küçük harf duyarlılığını devre dışı bırakabilirsiniz:
```php
use Illuminate\Support\Str;

$matches = Str::is('*.jpg', 'photo.JPG', ignoreCase: true);

// true
```

#### `Str::isAscii()` Metodu
`Str::isAscii` metodu, verilen bir dizenin 7 bit ASCII olup olmadığını belirler:
```php
use Illuminate\Support\Str;

$isAscii = Str::isAscii('Taylor');

// true

$isAscii = Str::isAscii('ü');

// false
```

#### `Str::isJson()` Metodu
`Str::isJson` metodu, verilen dizenin geçerli bir JSON olup olmadığını belirler:
```php
use Illuminate\Support\Str;

$result = Str::isJson('[1,2,3]');

// true

$result = Str::isJson('{"first": "John", "last": "Doe"}');

// true

$result = Str::isJson('{first: "John", last: "Doe"}');

// false
```

#### `Str::isUrl()` Metodu
`Str::isUrl` metodu, verilen dizenin geçerli bir URL olup olmadığını belirler:
```php
use Illuminate\Support\Str;

$isUrl = Str::isUrl('http://example.com');

// true

$isUrl = Str::isUrl('laravel');

// false
```
`isUrl` metodu çok çeşitli protokolleri geçerli olarak kabul eder. Ancak, `isUrl` metoduna sağlayarak geçerli kabul edilmesi gereken protokolleri belirtebilirsiniz:
```php
$isUrl = Str::isUrl('http://example.com', ['http', 'https']);
```

#### `Str::isUlid()` Metodu
`Str::isUlid` metodu, verilen dizenin geçerli bir ULID (Universally Unique Lexicographically Sortable Identifier) olup olmadığını belirler:
```php
use Illuminate\Support\Str;

$isUlid = Str::isUlid('01gd6r360bp37zj17nxb55yv40');

// true

$isUlid = Str::isUlid('laravel');

// false
```