# Laravel 12 Dokümantasyonu: Yerelleştirme (Localization)

## Giriş

Varsayılan olarak, Laravel uygulama iskeleti (skeleton) `lang` dizinini içermez. Laravel'in dil dosyalarını özelleştirmek isterseniz, bunları `lang:publish` Artisan komutu aracılığıyla yayınlayabilirsiniz.

Laravel'in yerelleştirme (localization) özellikleri, çeşitli dillerdeki dizeleri (strings) almak için kullanışlı bir yol sağlayarak uygulamanızda birden çok dili kolayca desteklemenize olanak tanır.

Laravel, çeviri dizelerini (translation strings) yönetmek için iki yol sağlar. İlk olarak, dil dizeleri uygulamanın `lang` dizini içindeki dosyalarda saklanabilir. Bu dizin içinde, uygulama tarafından desteklenen her dil için alt dizinler olabilir. Bu, Laravel'in doğrulama hata mesajları (validation error messages) gibi yerleşik Laravel özellikleri için çeviri dizelerini yönetmek üzere kullandığı yaklaşımdır:
```
/lang
    /en
        messages.php
    /es
        messages.php
```
Veya, çeviri dizeleri `lang` dizini içine yerleştirilen JSON dosyalarında tanımlanabilir. Bu yaklaşım benimsendiğinde, uygulamanız tarafından desteklenen her dilin bu dizin içinde karşılık gelen bir JSON dosyası olacaktır. Bu yaklaşım, çok sayıda çevrilebilir dizeye sahip uygulamalar için önerilir:
```
/lang
    en.json
    es.json
```
Bu dokümantasyonda, çeviri dizelerini yönetmek için her iki yaklaşımı da tartışacağız.

### Dil Dosyalarını Yayınlama (Publishing the Language Files)
Varsayılan olarak, Laravel uygulama iskeleti `lang` dizinini içermez. Laravel'in dil dosyalarını özelleştirmek veya kendi dosyalarınızı oluşturmak isterseniz, `lang:publish` Artisan komutu aracılığıyla `lang` dizinini oluşturmalısınız (scaffold). `lang:publish` komutu, uygulamanızda `lang` dizinini oluşturacak ve Laravel tarafından kullanılan varsayılan dil dosyası setini yayınlayacaktır:
```bash
php artisan lang:publish
```

### Yerel Ayarı (Locale) Yapılandırma (Configuring the Locale)
Uygulamanız için varsayılan dil, `config/app.php` yapılandırma dosyasındaki `locale` yapılandırma seçeneğinde saklanır ve bu genellikle `APP_LOCALE` ortam değişkeni kullanılarak ayarlanır. Bu değeri uygulamanızın ihtiyaçlarına uyacak şekilde değiştirmekte özgürsünüz.

Ayrıca, varsayılan dil belirli bir çeviri dizesini içermediğinde kullanılacak bir "yedek dil" (fallback language) de yapılandırabilirsiniz. Varsayılan dil gibi, yedek dil de `config/app.php` yapılandırma dosyasında yapılandırılır ve değeri tipik olarak `APP_FALLBACK_LOCALE` ortam değişkeni kullanılarak ayarlanır.

`App` facade'ı tarafından sağlanan `setLocale` metodunu kullanarak tek bir HTTP isteği için varsayılan dili çalışma zamanında değiştirebilirsiniz:
```php
use Illuminate\Support\Facades\App;

Route::get('/greeting/{locale}', function (string $locale) {
    if (! in_array($locale, ['en', 'es', 'fr'])) {
        abort(400);
    }

    App::setLocale($locale);

    // ...
});
```

#### Geçerli Yerel Ayarı Belirleme (Determining the Current Locale)
Geçerli yerel ayarı belirlemek veya yerel ayarın belirli bir değer olup olmadığını kontrol etmek için `App` facade'ı üzerindeki `currentLocale` ve `isLocale` metotlarını kullanabilirsiniz:
```php
use Illuminate\Support\Facades\App;

$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

### Çoğullaştırma (Pluralization) Dili
Eloquent ve framework'ün diğer bölümleri tarafından tekil dizeleri çoğul dizelere dönüştürmek için kullanılan Laravel'in "çoğullaştırıcısını" (pluralizer), İngilizce dışında bir dil kullanacak şekilde yönlendirebilirsiniz. Bu, uygulamanızın servis sağlayıcılarından birinin `boot` metodu içinde `useLanguage` metodu çağrılarak gerçekleştirilebilir. Çoğullaştırıcının şu anda desteklediği diller şunlardır: `french`, `norwegian-bokmal`, `portuguese`, `spanish` ve `turkish`:
```php
use Illuminate\Support\Pluralizer;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Pluralizer::useLanguage('spanish');

    // ...
}
```
Çoğullaştırıcının dilini özelleştirirseniz, Eloquent modelinizin tablo adlarını açıkça tanımlamalısınız.

## Çeviri Dizeleri Tanımlama (Defining Translation Strings)

### Kısa Anahtarları Kullanma (Using Short Keys)
Tipik olarak, çeviri dizeleri `lang` dizini içindeki dosyalarda saklanır. Bu dizin içinde, uygulamanız tarafından desteklenen her dil için bir alt dizin bulunmalıdır. Bu, Laravel'in doğrulama hata mesajları gibi yerleşik Laravel özellikleri için çeviri dizelerini yönetmek üzere kullandığı yaklaşımdır:
```
/lang
    /en
        messages.php
    /es
        messages.php
```
Tüm dil dosyaları, anahtarlanmış dizelerden oluşan bir dizi (array of keyed strings) döndürür. Örneğin:
```php
<?php

// lang/en/messages.php

return [
    'welcome' => 'Welcome to our application!',
];
```
Bölgeye göre farklılık gösteren diller için, dil dizinlerini ISO 15897'ye göre adlandırmalısınız. Örneğin, "en-gb" yerine "en_GB" kullanılmalıdır.

### Çeviri Dizelerini Anahtar Olarak Kullanma (Using Translation Strings as Keys)
Çok sayıda çevrilebilir dizeye sahip uygulamalar için, her dizeyi bir "kısa anahtar" (short key) ile tanımlamak, görünümlerinizde (views) anahtarlara başvururken kafa karıştırıcı hale gelebilir ve uygulamanız tarafından desteklenen her çeviri dizesi için sürekli olarak anahtarlar icat etmek zahmetlidir.

Bu nedenle Laravel, çeviri dizelerini tanımlamak için dizenin "varsayılan" çevirisini anahtar olarak kullanma desteği de sağlar. Çeviri dizelerini anahtar olarak kullanan dil dosyaları, `lang` dizininde JSON dosyaları olarak saklanır. Örneğin, uygulamanızın bir İspanyolca çevirisi varsa, bir `lang/es.json` dosyası oluşturmalısınız:
```json
{
    "I love programming.": "Me encanta programar."
}
```

#### Anahtar / Dosya Çakışmaları (Key / File Conflicts)
Diğer çeviri dosya adlarıyla çakışan çeviri dizesi anahtarları tanımlamamalısınız. Örneğin, `__('Action')` ifadesini "NL" yerel ayarı için çevirirken, bir `nl/action.php` dosyası mevcut ancak `nl.json` dosyası mevcut değilse, çevirmen (translator) `nl/action.php` dosyasının tüm içeriğini döndürecektir.

## Çeviri Dizelerini Alma (Retrieving Translation Strings)

Çeviri dizelerinizi dil dosyalarından `__` yardımcı fonksiyonunu kullanarak alabilirsiniz. Çeviri dizelerinizi tanımlamak için "kısa anahtarlar" kullanıyorsanız, anahtarı içeren dosyayı ve anahtarın kendisini "nokta" sözdizimini ("dot" syntax) kullanarak `__` fonksiyonuna iletmelisiniz. Örneğin, `lang/en/messages.php` dil dosyasından `welcome` çeviri dizesini alalım:
```php
echo __('messages.welcome');
```
Belirtilen çeviri dizesi mevcut değilse, `__` fonksiyonu çeviri dizesi anahtarını döndürecektir. Yani, yukarıdaki örneği kullanırsak, çeviri dizesi mevcut değilse `__` fonksiyonu `messages.welcome` döndürecektir.

Varsayılan çeviri dizelerinizi çeviri anahtarlarınız olarak kullanıyorsanız, dizenizin varsayılan çevirisini `__` fonksiyonuna iletmelisiniz:
```php
echo __('I love programming.');
```
Yine, çeviri dizesi mevcut değilse, `__` fonksiyonu kendisine verilen çeviri dizesi anahtarını döndürecektir.

Blade şablonlama motorunu kullanıyorsanız, çeviri dizesini görüntülemek için `{{ }}` echo sözdizimini kullanabilirsiniz:
```blade
{{ __('messages.welcome') }}
```

### Çeviri Dizelerinde Parametreleri Değiştirme (Replacing Parameters in Translation Strings)
İsterseniz, çeviri dizelerinizde yer tutucular (placeholders) tanımlayabilirsiniz. Tüm yer tutucular bir `:` ile ön eklenir (prefixed). Örneğin, bir `name` yer tutucusu olan bir karşılama mesajı tanımlayabilirsiniz:
```php
'welcome' => 'Welcome, :name',
```
Bir çeviri dizesini alırken yer tutucuları değiştirmek için, `__` fonksiyonuna ikinci argüman olarak bir değiştirme dizisi (array of replacements) iletebilirsiniz:
```php
echo __('messages.welcome', ['name' => 'dayle']);
```
Yer tutucunuz tamamen büyük harflerden oluşuyorsa veya yalnızca ilk harfi büyükse, çevrilen değer buna uygun olarak büyük harfle yazılacaktır:
```php
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

#### Nesne Değiştirme Biçimlendirmesi (Object Replacement Formatting)
Bir nesneyi çeviri yer tutucusu olarak sağlamaya çalışırsanız, nesnenin `__toString` metodu çağrılacaktır. `__toString` metodu, PHP'nin yerleşik "sihirli metotlarından" (magic methods) biridir. Ancak, bazen belirli bir sınıfın `__toString` metodu üzerinde kontrolünüz olmayabilir, örneğin etkileşimde bulunduğunuz sınıf üçüncü taraf bir kütüphaneye aitse.

Bu durumlarda, Laravel bu belirli nesne türü için özel bir biçimlendirme işleyicisi (custom formatting handler) kaydetmenize izin verir. Bunu başarmak için çevirmenin (translator) `stringable` metodunu çağırmalısınız. `stringable` metodu, biçimlendirmeden sorumlu olduğu nesne türünü tip belirtmesi (type-hint) gereken bir closure kabul eder. Tipik olarak, `stringable` metodu uygulamanızın `AppServiceProvider` sınıfının `boot` metodu içinde çağrılmalıdır:
```php
use Illuminate\Support\Facades\Lang;
use Money\Money;

/**
 * Herhangi bir uygulama servisini önyükle (bootstrap).
 */
public function boot(): void
{
    Lang::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

### Çoğullaştırma (Pluralization)
Farklı dillerin çoğullaştırma için çeşitli karmaşık kuralları olduğundan, çoğullaştırma karmaşık bir problemdir; ancak Laravel, tanımladığınız çoğullaştırma kurallarına göre dizeleri farklı şekilde çevirmenize yardımcı olabilir. Bir `|` karakteri kullanarak, bir dizenin tekil ve çoğul biçimlerini ayırt edebilirsiniz:
```php
'apples' => 'There is one apple|There are many apples',
```
Elbette, çeviri dizeleri anahtar olarak kullanıldığında da çoğullaştırma desteklenir:
```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```
Ayrıca, birden çok değer aralığı için çeviri dizeleri belirten daha karmaşık çoğullaştırma kuralları da oluşturabilirsiniz:
```php
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
```
Çoğullaştırma seçeneklerine sahip bir çeviri dizesi tanımladıktan sonra, belirli bir "sayı" (count) için satırı almak üzere `trans_choice` fonksiyonunu kullanabilirsiniz. Bu örnekte, sayı birden büyük olduğu için çeviri dizesinin çoğul biçimi döndürülür:
```php
echo trans_choice('messages.apples', 10);
```
Ayrıca çoğullaştırma dizelerinde yer tutucu nitelikleri de tanımlayabilirsiniz. Bu yer tutucular, `trans_choice` fonksiyonuna üçüncü argüman olarak bir dizi iletilerek değiştirilebilir:
```php
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```
`trans_choice` fonksiyonuna iletilen tamsayı değerini görüntülemek isterseniz, yerleşik `:count` yer tutucusunu kullanabilirsiniz:
```php
'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',
```

## Paket Dil Dosyalarını Geçersiz Kılma (Overriding Package Language Files)

Bazı paketler kendi dil dosyalarıyla birlikte gelebilir. Bu satırları düzeltmek için paketin temel dosyalarını değiştirmek yerine, `lang/vendor/{package}/{locale}` dizinine dosyalar yerleştirerek bunları geçersiz kılabilirsiniz (override).

Örneğin, `skyrim/hearthfire` adlı bir paket için `messages.php` dosyasındaki İngilizce çeviri dizelerini geçersiz kılmanız gerekiyorsa, şu konuma bir dil dosyası yerleştirmelisiniz: `lang/vendor/hearthfire/en/messages.php`. Bu dosya içinde, yalnızca geçersiz kılmak istediğiniz çeviri dizelerini tanımlamalısınız. Geçersiz kılmadığınız çeviri dizeleri yine de paketin orijinal dil dosyalarından yüklenecektir.