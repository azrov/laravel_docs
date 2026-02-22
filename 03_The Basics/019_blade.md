# Laravel 12 Dokümantasyonu: Blade Şablonları (Blade Templates)

## Giriş

Blade, Laravel ile birlikte gelen basit ama güçlü şablonlama motorudur (templating engine). Bazı PHP şablonlama motorlarının aksine Blade, şablonlarınızda düz PHP kodu kullanmanızı kısıtlamaz. Aslında, tüm Blade şablonları derlenir (compiled) ve değiştirilene kadar önbellekte (cache) tutulur; bu da Blade'in uygulamanıza neredeyse hiç ek yük (overhead) getirmediği anlamına gelir. Blade şablon dosyaları `.blade.php` dosya uzantısını kullanır ve tipik olarak `resources/views` dizininde saklanır.

Blade görünümleri, global `view` yardımcısı kullanılarak rotalardan veya controller'lardan döndürülebilir. Elbette, görünümlerle ilgili dokümantasyonda belirtildiği gibi, `view` yardımcısının ikinci argümanı kullanılarak Blade görünümüne veri aktarılabilir:
```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

### Blade'i Livewire ile Süper Güçlendirme (Supercharging Blade With Livewire)
Blade şablonlarınızı bir sonraki seviyeye taşımak ve kolaylıkla dinamik arayüzler oluşturmak ister misiniz? Laravel Livewire'a göz atın. Livewire, tipik olarak yalnızca React, Vue veya Svelte gibi ön yüz framework'leriyle mümkün olan dinamik işlevsellikle güçlendirilmiş Blade bileşenleri yazmanıza olanak tanır; böylece birçok JavaScript framework'ünün karmaşıklıkları, istemci tarafı render etme (client-side rendering) veya derleme adımları (build steps) olmadan modern, tepkisel (reactive) ön yüzler oluşturmak için harika bir yaklaşım sağlar.

## Veri Görüntüleme (Displaying Data)

Değişkeni kaşlı ayraçlar (curly braces) içine alarak Blade görünümlerinize aktarılan verileri görüntüleyebilirsiniz. Örneğin, aşağıdaki rota verilmişse:
```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```
`name` değişkeninin içeriğini şu şekilde görüntüleyebilirsiniz:
```blade
Hello, {{ $name }}.
```
Blade'in `{{ }}` echo ifadeleri, XSS saldırılarını önlemek için otomatik olarak PHP'nin `htmlspecialchars` fonksiyonundan geçirilir.

Görünüme aktarılan değişkenlerin içeriğini görüntülemekle sınırlı değilsiniz. Herhangi bir PHP fonksiyonunun sonuçlarını da görüntüleyebilirsiniz. Aslında, bir Blade echo ifadesinin içine istediğiniz herhangi bir PHP kodunu koyabilirsiniz:
```blade
The current UNIX timestamp is {{ time() }}.
```

### HTML Varlık Kodlaması (HTML Entity Encoding)
Varsayılan olarak, Blade (ve Laravel'in `e` fonksiyonu) HTML varlıklarını (entities) çift kodlayacaktır (double encode). Çift kodlamayı devre dışı bırakmak isterseniz, `AppServiceProvider`'ınızın `boot` metodundan `Blade::withoutDoubleEncoding` metodunu çağırın:
```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Herhangi bir uygulama servisini önyükle (bootstrap).
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

#### Kaçışsız (Unescaped) Veri Görüntüleme
Varsayılan olarak, Blade `{{ }}` ifadeleri, XSS saldırılarını önlemek için otomatik olarak PHP'nin `htmlspecialchars` fonksiyonundan geçirilir. Verilerinizin kaçışa uğramasını (escaped) istemiyorsanız, aşağıdaki sözdizimini kullanabilirsiniz:
```blade
Hello, {!! $name !!}.
```
Uygulamanızın kullanıcıları tarafından sağlanan içeriği görüntülerken çok dikkatli olun. Kullanıcı tarafından sağlanan verileri görüntülerken XSS saldırılarını önlemek için genellikle kaçışlı (escaped), çift kaşlı ayraç sözdizimini kullanmalısınız.

### Blade ve JavaScript Framework'leri (Blade and JavaScript Frameworks)
Birçok JavaScript framework'ü, belirli bir ifadenin tarayıcıda görüntülenmesi gerektiğini belirtmek için "kaşlı ayraçlar" kullandığından, bir ifadenin olduğu gibi kalması gerektiğini Blade render motoruna bildirmek için `@` sembolünü kullanabilirsiniz. Örneğin:
```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```
Bu örnekte, `@` sembolü Blade tarafından kaldırılacaktır; ancak, `{{ name }}` ifadesi Blade motoru tarafından olduğu gibi bırakılacak ve JavaScript framework'ünüz tarafından render edilmesine izin verilecektir.

`@` sembolü, Blade direktiflerinden kaçmak için de kullanılabilir:
```blade
{{-- Blade şablonu --}}
@@if()

<!-- HTML çıktısı -->
@if()
```

#### JSON Render Etme (Rendering JSON)
Bazen bir JavaScript değişkenini başlatmak amacıyla bir diziyi JSON olarak render etmek için görünümünüze aktarabilirsiniz. Örneğin:
```blade
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```
Ancak, manuel olarak `json_encode` çağırmak yerine `Illuminate\Support\Js::from` metodunu kullanabilirsiniz. `from` metodu, PHP'nin `json_encode` fonksiyonuyla aynı argümanları kabul eder; ancak, ortaya çıkan JSON'ın HTML tırnakları içine dahil edilmek üzere uygun şekilde kaçışa uğratılmasını (escaped) sağlayacaktır. `from` metodu, verilen nesneyi veya diziyi geçerli bir JavaScript nesnesine dönüştürecek bir `JSON.parse` JavaScript ifadesi döndürecektir:
```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```
Laravel uygulama iskeletinin (skeleton) en son sürümleri, Blade şablonlarınız içinde bu işlevselliğe uygun erişim sağlayan bir `Js` facade'ı içerir:
```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```
`Js::from` metodunu yalnızca mevcut değişkenleri JSON olarak render etmek için kullanmalısınız. Blade şablonlama düzenli ifadelere (regular expressions) dayanır ve direktife karmaşık bir ifade iletmeye çalışmak beklenmeyen hatalara neden olabilir.

#### @verbatim Direktifi
Şablonunuzun büyük bir bölümünde JavaScript değişkenleri görüntülüyorsanız, her Blade echo ifadesine bir `@` sembolü eklemek zorunda kalmamak için HTML'i `@verbatim` direktifi içine sarabilirsiniz:
```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Blade Direktifleri (Blade Directives)

Şablon kalıtımı (template inheritance) ve veri görüntülemenin yanı sıra Blade, koşullu ifadeler (conditional statements) ve döngüler (loops) gibi yaygın PHP kontrol yapıları için kullanışlı kısayollar da sağlar. Bu kısayollar, PHP karşılıklarına aşina kalırken PHP kontrol yapılarıyla çalışmanın çok temiz ve kısa bir yolunu sunar.

### If İfadeleri (If Statements)
`@if`, `@elseif`, `@else` ve `@endif` direktiflerini kullanarak `if` ifadeleri oluşturabilirsiniz. Bu direktifler, PHP karşılıklarıyla aynı şekilde çalışır:
```blade
@if (count($records) === 1)
    Bir kaydım var!
@elseif (count($records) > 1)
    Birden çok kaydım var!
@else
    Hiç kaydım yok!
@endif
```
Kolaylık olması açısından, Blade ayrıca bir `@unless` direktifi de sağlar:
```blade
@unless (Auth::check())
    Oturum açmamışsınız.
@endunless
```
Daha önce tartışılan koşullu direktiflere ek olarak, `@isset` ve `@empty` direktifleri, ilgili PHP fonksiyonları için kullanışlı kısayollar olarak kullanılabilir:
```blade
@isset($records)
    // $records tanımlı ve null değil...
@endisset

@empty($records)
    // $records "boş"...
@endempty
```

#### Kimlik Doğrulama (Authentication) Direktifleri
`@auth` ve `@guest` direktifleri, geçerli kullanıcının kimlik doğrulamasının yapılıp yapılmadığını veya bir misafir (guest) olup olmadığını hızlıca belirlemek için kullanılabilir:
```blade
@auth
    // Kullanıcının kimliği doğrulandı...
@endauth

@guest
    // Kullanıcının kimliği doğrulanmadı...
@endguest
```
Gerekirse, `@auth` ve `@guest` direktiflerini kullanırken kontrol edilmesi gereken kimlik doğrulama koruyucusunu (guard) belirtebilirsiniz:
```blade
@auth('admin')
    // Kullanıcının kimliği doğrulandı...
@endauth

@guest('admin')
    // Kullanıcının kimliği doğrulanmadı...
@endguest
```

#### Ortam (Environment) Direktifleri
`@production` direktifini kullanarak uygulamanın üretim (production) ortamında çalışıp çalışmadığını kontrol edebilirsiniz:
```blade
@production
    // Üretime özgü içerik...
@endproduction
```
Veya, `@env` direktifini kullanarak uygulamanın belirli bir ortamda çalışıp çalışmadığını belirleyebilirsiniz:
```blade
@env('staging')
    // Uygulama "staging" ortamında çalışıyor...
@endenv

@env(['staging', 'production'])
    // Uygulama "staging" veya "production" ortamında çalışıyor...
@endenv
```

#### Bölüm (Section) Direktifleri
`@hasSection` direktifini kullanarak bir şablon kalıtım bölümünün (section) içeriğe sahip olup olmadığını belirleyebilirsiniz:
```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```
Bir bölümün içeriğe sahip olmadığını belirlemek için `sectionMissing` direktifini kullanabilirsiniz:
```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

#### Oturum (Session) Direktifleri
`@session` direktifi, bir oturum (session) değerinin var olup olmadığını belirlemek için kullanılabilir. Oturum değeri varsa, `@session` ve `@endsession` direktifleri arasındaki şablon içeriği değerlendirilecektir. `@session` direktifinin içeriğinde, oturum değerini görüntülemek için `$value` değişkenini görüntüleyebilirsiniz:
```blade
@session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
@endsession
```

#### Bağlam (Context) Direktifleri
`@context` direktifi, bir bağlam (context) değerinin var olup olmadığını belirlemek için kullanılabilir. Bağlam değeri varsa, `@context` ve `@endcontext` direktifleri arasındaki şablon içeriği değerlendirilecektir. `@context` direktifinin içeriğinde, bağlam değerini görüntülemek için `$value` değişkenini görüntüleyebilirsiniz:
```blade
@context('canonical')
    <link href="{{ $value }}" rel="canonical">
@endcontext
```

### Switch İfadeleri (Switch Statements)
Switch ifadeleri, `@switch`, `@case`, `@break`, `@default` ve `@endswitch` direktifleri kullanılarak oluşturulabilir:
```blade
@switch($i)
    @case(1)
        İlk durum...
        @break

    @case(2)
        İkinci durum...
        @break

    @default
        Varsayılan durum...
@endswitch
```

### Döngüler (Loops)
Koşullu ifadelere ek olarak Blade, PHP'nin döngü yapılarıyla çalışmak için basit direktifler sağlar. Yine, bu direktiflerin her biri PHP karşılıklarıyla aynı şekilde çalışır:
```blade
@for ($i = 0; $i < 10; $i++)
    Geçerli değer {{ $i }}
@endfor

@foreach ($users as $user)
    <p>Bu kullanıcı {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>Hiç kullanıcı yok</p>
@endforelse

@while (true)
    <p>Sonsuza kadar döngü yapıyorum.</p>
@endwhile
```
Bir `foreach` döngüsünde yineleme yaparken, döngüdeki ilk veya son yinelemede olup olmadığınız gibi döngü hakkında değerli bilgiler elde etmek için `$loop` değişkenini kullanabilirsiniz.

Döngüleri kullanırken, `@continue` ve `@break` direktiflerini kullanarak geçerli yinelemeyi atlayabilir veya döngüyü sonlandırabilirsiniz:
```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```
Devam etme veya sonlandırma koşulunu doğrudan direktif bildiriminin içine de dahil edebilirsiniz:
```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

### Döngü Değişkeni (The Loop Variable)
Bir `foreach` döngüsünde yineleme yaparken, döngünüzün içinde bir `$loop` değişkeni kullanılabilir olacaktır. Bu değişken, geçerli döngü indeksi (index) ve döngüdeki ilk veya son yineleme olup olmadığı gibi bazı yararlı bilgilere erişim sağlar:
```blade
@foreach ($users as $user)
    @if ($loop->first)
        Bu ilk yineleme.
    @endif

    @if ($loop->last)
        Bu son yineleme.
    @endif

    <p>Bu kullanıcı {{ $user->id }}</p>
@endforeach
```
İç içe bir döngüdeyseniz, üst döngünün `$loop` değişkenine `parent` özelliği aracılığıyla erişebilirsiniz:
```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            Bu, üst döngünün ilk yinelemesi.
        @endif
    @endforeach
@endforeach
```
`$loop` değişkeni ayrıca çeşitli başka kullanışlı özellikler de içerir:

| Özellik | Açıklama |
| :--- | :--- |
| `$loop->index` | Geçerli döngü yinelemesinin indeksi (0'dan başlar). |
| `$loop->iteration` | Geçerli döngü yinelemesi (1'den başlar). |
| `$loop->remaining` | Döngüde kalan yineleme sayısı. |
| `$loop->count` | Yineleme yapılan dizideki toplam öğe sayısı. |
| `$loop->first` | Döngüdeki ilk yineleme olup olmadığı. |
| `$loop->last` | Döngüdeki son yineleme olup olmadığı. |
| `$loop->even` | Döngüdeki çift sayılı yineleme olup olmadığı. |
| `$loop->odd` | Döngüdeki tek sayılı yineleme olup olmadığı. |
| `$loop->depth` | Geçerli döngünün iç içelik seviyesi. |
| `$loop->parent` | İç içe bir döngüde, üst döngünün `$loop` değişkeni. |

### Koşullu Sınıflar (Conditional Classes) ve Stiller (Styles)
`@class` direktifi, koşullu olarak bir CSS sınıfı dizesi derler. Direktif, dizi anahtarının eklemek istediğiniz sınıfı veya sınıfları içerdiği, değerin ise bir boolean ifade olduğu bir sınıflar dizisi kabul eder. Dizi öğesinin sayısal bir anahtarı varsa, her zaman render edilen sınıf listesine dahil edilecektir:
```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```
Benzer şekilde, `@style` direktifi, bir HTML öğesine koşullu olarak satır içi (inline) CSS stilleri eklemek için kullanılabilir:
```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

### Ek Nitelikler (Additional Attributes)
Kolaylık olması açısından, belirli bir HTML onay kutusu (checkbox) girişinin "işaretli" (checked) olup olmadığını kolayca belirtmek için `@checked` direktifini kullanabilirsiniz. Bu direktif, sağlanan koşul `true` olarak değerlendirilirse `checked` ifadesini görüntüleyecektir:
```blade
<input
    type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active))
/>
```
Benzer şekilde, `@selected` direktifi, belirli bir seçim (select) seçeneğinin "seçili" (selected) olup olmadığını belirtmek için kullanılabilir:
```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```
Ayrıca, `@disabled` direktifi, belirli bir öğenin "devre dışı" (disabled) olup olmadığını belirtmek için kullanılabilir:
```blade
<button type="submit" @disabled($errors->isNotEmpty())>Gönder</button>
```
Dahası, `@readonly` direktifi, belirli bir öğenin "salt okunur" (readonly) olup olmadığını belirtmek için kullanılabilir:
```blade
<input
    type="email"
    name="email"
    value="email@laravel.com"
    @readonly($user->isNotAdmin())
/>
```
Ek olarak, `@required` direktifi, belirli bir öğenin "gerekli" (required) olup olmadığını belirtmek için kullanılabilir:
```blade
<input
    type="text"
    name="title"
    value="title"
    @required($user->isAdmin())
/>
```

### Alt Görünümleri Dahil Etme (Including Subviews)
`@include` direktifini kullanmakta özgür olsanız da, Blade bileşenleri (components) benzer işlevsellik sağlar ve birkaç avantaj sunar. Devamı bir sonraki bölümde...