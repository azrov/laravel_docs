# Laravel Değişiklik Günlüğü (Changelog)

Laravel'in açık kaynaklı ürünlerindeki yeni güncellemeler ve iyileştirmeler.

## Ocak 2026

## Laravel Framework 12.x

### [12.x] İzole Blade Eklentileri için `@includeIsolated` Direktifi Eklendi
**Katkıda bulunan:** @KennedyTedesco

Bir parçayı (partial) yeniden kullanmanız gerektiğinde, değişkenlerin yanlışlıkla içeri veya dışarı sızmasını önlemek için `@includeIsolated` size temiz ve öngörülebilir bir sınır sağlar. Özellikle paylaşılan UI bileşenleri ve "gizemli verilerin" hata ayıklamasının maliyetli olabileceği e-postalar için harikadır.
```blade
{{-- Yalnızca açıkça ilettiğiniz değişkenleri alır --}}
@includeIsolated("partials.alert", ["type" => "success", "message" => $message])
```

### [12.x] `Cache::withoutOverlapping()` Metodu Eklendi
**Katkıda bulunan:** @mathiasgrimm

Bu, önbellek kilitleri (cache locks) etrafında düzenli ve anlamlı bir sarmalayıcı (wrapper) ekleyerek daha az tekrar eden kod (boilerplate) ile örtüşen işleri (overlapping work) önlemenizi sağlar. Cron işleri, rapor oluşturma ve "bir seferde yalnızca bir tane" çalışması gereken görevler için mükemmeldir.
```php
Cache::withoutOverlapping('reports:daily')
    ->block(10, function () {
        // Bu iş eşzamanlı olarak çalışmayacak.
        GenerateDailyReport::run();
    });
```

### Veritabanında Vektör Yerleştirmeleri (Vector Embeddings) Desteği
**Katkıda bulunan:** @taylorotwell

Bu çalışma, Laravel'in "vektör" yeteneklerini ileriye taşıyarak, yerleştirmeler (embeddings) ve benzerlik araması (similarity search) ile uğraşırken daha zengin entegrasyonlar ve daha akıcı iş akışları için zemin hazırlıyor. Yapay zeka destekli özellikleri keşfediyorsanız, bu değişiklik tam size göre.

### [12.x] `Collection::containsManyItems()` Metodu Eklendi
**Katkıda bulunan:** @stevebauman

Mevcut `containsOneItem` metodunu tamamlamak için yeni bir `containsManyItems` metodu eklendi. Bu, bir koleksiyonun iki veya daha fazla öğe içerip içermediğini hızlıca kontrol etmenizi sağlar.
```php
match (true) {
    $collection->isEmpty() => '...',
    $collection->containsOneItem() => '...',
    $collection->containsManyItems() => '...',
};
```

### [12.x] JSON:API Kaynağı (Resource)
**Katkıda bulunan:** @crynobone

Bu, JSON:API kaynak desteğini tanıtarak, ekipler ve servisler arasında tutarlı bir yapıya sahip, standartlara uygun API'ler sunmayı kolaylaştırır. Yanıtları standartlaştırıyorsanız (veya JSON:API bekleyen istemcilerle entegre oluyorsanız), bu, özel biçimlendirmeyi azaltır ve API yüzeyinizi daha öngörülebilir hale getirir.

### [12.x] Oturum Anahtarları (Session Keys) için `BackedEnum` Desteği Eklendi
**Katkıda bulunan:** @ahinkle

Artık oturum anahtarları olarak desteklenmiş enum'ları (backed enums) kullanabilirsiniz. Bu, anahtar adlarını merkezileştirir ve yazım hatasını önler. Özellikle oturum anahtarlarının zamanla dağılma eğiliminde olduğu büyük uygulamalarda faydalıdır.
```php
enum CheckoutSession: string
{
    case Cart = 'checkout.cart';
    case ShippingAddress = 'checkout.shipping_address';
    case PaymentMethod = 'checkout.payment_method';
}

session()->put(CheckoutSession::Cart, $items);
session()->get(CheckoutSession::Cart);
```

### [12.x] Önbellek Anahtarları (Cache Keys) için `BackedEnum` Kullanımına İzin Verildi
**Katkıda bulunan:** @jackbayliss

Aynı üretkenlik artışı, şimdi de önbellek için: desteklenmiş enum'lar önbellek anahtarları olarak kullanılabilir. Bu, anahtar adlandırmasını standartlaştırmanıza ve ince çakışmaları önlemenize yardımcı olur. Önbellek tanımlayıcıları için tek bir doğruluk kaynağı (single source of truth) isteyen ekipler için harika bir uyumdur.
```php
enum CacheKey: string
{
    case Settings = 'settings.global';
}

Cache::put(CacheKey::Settings, $settings, now()->addHour());

$settings = Cache::get(CacheKey::Settings);
```

### [12.x] `env:encrypt` Komutuna Görünür Anahtar Adları için `--readable` Bayrağı Eklendi
**Katkıda bulunan:** @mathiasgrimm

Pratik bir yaşam kalitesi iyileştirmesi: `--readable` bayrağı, şifrelenmiş env çıktısını denetlemeyi kolaylaştırır. Anahtar adlarını görünür tutar, bu da incelemeler (reviews) ve olay müdahalesi (incident response) sırasında yardımcı olurken gizli değerleri korur. Dağıtımları sağlamlaştırıyorsanız, bu, Laravel'in ortam şifreleme iş akışıyla iyi bir uyum sağlar.
```bash
php artisan env:encrypt --readable
```

## Inertia

### `cancelAll` Metoduna `async` ve `sync` Seçenekleri Eklendi
**Katkıda bulunan:** @pascalbaljet

Kullanımdan kaldırılan (deprecated) `router.cancel()` işlevinin yerini alan `router.cancelAll({ async: false })` ile aynı işlevselliği elde edebilirsiniz. İstek iptali üzerinde daha fazla kontrol sağlamak, yoğun gezinme veya hızlı girdi altında daha akıcı bir kullanıcı deneyimi (UX) anlamına gelir.

### [2.x] Form Bağlamı (Form Context) Desteği Eklendi
**Katkıda bulunan:** @laserhybiz

Yeni `useFormContext` hook'u, alt bileşenlerin, gereksiz prop geçirme (prop drilling) işlemleri yapmak zorunda kalmadan üst bileşen formları hakkında bilgi sahibi olmasını sağlar.
```vue
<!-- Parent.vue -->
<template>
    <Form action="/users" method="post">
        <Input name="name" />
    </Form>
</template>

<!-- Input.vue -->
<script setup>
import { useFormContext } from "@inertiajs/vue3";

defineProps({
    name: {
        type: String,
        required: true,
    },
});

const form = useFormContext();
</script>

<template>
    <input type="text" :name="name" />

    <div v-if="form.errors[name]">
        {{ form.errors[name] }}
    </div>
</template>
```

### Form Yardımcısına `dontRemember()` Metodu Eklendi
**Katkıda bulunan:** @pascalbaljet

Bu, istemediğiniz durumlarda Inertia'nın form hatırlama (form remembering) davranışını devre dışı bırakmanın basit bir yolunu sunar. Hassas formlar veya eski değerlerin kalmasını istemediğiniz tek seferlik akışlar (örneğin, parola sıfırlama, davet akışları) için kullanışlıdır.

## Başlangıç Kitleri (Starter Kits)

Bu ay tüm Başlangıç Kitlerimizde bir dizi güncelleme yaptık! Çeşitli ayarlamalar arasında:

### `AppServiceProvider`a Daha Güvenli Varsayılanlar Eklendi
**Katkıda bulunan:** @WendellAdriel

`AppServiceProvider`daki daha güvenli varsayılanlar, yeni projelerde "hata yapma" faktörünü azaltır. Kiti hafif ve özelleştirmesi kolay tutarken daha güvenli, üretime hazır bir başlangıç noktasıyla başlamanıza yardımcı olur.
*   `CarbonImmutable` ile değişmez (immutable) tarihler zorunlu kılınıyor
*   Üretimde yıkıcı komutlara izin verilmiyor
*   Üretimde varsayılan olarak daha güvenli parolalar gerekiyor

### Livewire 4 Desteği
**Katkıda bulunan:** @calebporzio

Livewire başlangıç kitleri artık Livewire 4 için hazır. Böylece yeni projelerinize en son Livewire temeli üzerinde başlayabilir, yükseltme işleriyle zaman harcamazsınız. Yeni bir uygulama planlıyorsanız, bu, başlangıç çizginizi güncel tutar.

### Linting için Pint Yapılandırma Dosyası ve Util Composer Betikleri Eklendi
**Katkıda bulunan:** @WendellAdriel

Kutudan çıkar çıkmaz hazır gelen linting, ekipleri hızlandırır: tutarlı biçimlendirme, kod incelemesinde daha az stil takıntısı ve daha kolay ekip içi uyum (onboarding). Dahil edilen Pint yapılandırması ve yardımcı betikler, "her şeyi biçimlendir" işlemini tek komutluk bir alışkanlık haline getirir.

## Boost

### Boost 2.0
Laravel Boost 2.0 ile Skills (Beceriler) özelliğini yayınladık. Bu, yapay zeka ajanlarının Laravel uygulamalarınızı anlama ve onlarla çalışma biçiminde büyük bir yükseltmedir.

Bu güncelleme, birçok paket yönergesini (guidelines) ajan becerilerine dönüştürerek önemli ölçüde daha iyi bağlam yönetimi (context management) sağlar. Pakete özgü bilgileri yalnızca alakalı olduğunda yükleyerek, ajanlar çok daha az bağlam gürültüsüyle daha doğru, daha yüksek kaliteli Laravel kodu üretir. Laravel uygulamaları oluştururken yapay zeka ajanlarından en iyi sonuçları almak için bu yükseltme olmazsa olmazdır.

Sonuç olarak, mevcut yönergeler yaklaşık yüzde 40 daha hafif hale geldi, bu da ajan yanıtlarını daha odaklı ve çok daha kaliteli yapıyor.

### Laravel Kod Basitleştirici (Code Simplifier) Komut İstemi (Prompt) Eklendi
**Katkıda bulunan:** @pushpak1300

Özel bir "kod basitleştirici" komut istemi, ayrıntılı veya tekrarlayan Laravel kodunu hızlıca temiz, deyimsel (idiomatic) kalıplara dönüştürmenize yardımcı olur. PR'ları cilalamak ve hızlı hareket ederken yeniden düzenlemeleri (refactor) hızlandırmak için harikadır.

### Livewire 4 Yükseltme Komut İstemi (Prompt) Eklendi
**Katkıda bulunan:** @pushpak1300

Odaklanmış bir kılavuzunuz olduğunda yükseltme yapmak daha kolaydır. Bu komut istemi, Livewire 4 için anahtar değişiklikleri belirlemenize ve bunlar üzerinde sistematik olarak çalışmanıza yardımcı olarak yükseltme riskini azaltır ve "bu neden bozuldu?" süresini kısaltır.

## Echo

### `useEcho`'daki Performans Sorunları ve Bayat Closure Hataları Çözüldü
**Katkıda bulunan:** @pataar

Bu, bayat closure (stale closure) sorunlarını ele alarak hook güvenilirliğini ve performansını artırır. Bu, daha az kafa karıştırıcı gerçek zamanlı UI hatası ve yoğun olay trafiği altında daha akıcı oluşturma (rendering) anlamına gelir. Tepkisel (reactive) panolar veya sohbet deneyimleri oluşturuyorsanız, bu, istemci tarafı davranışınızı daha güvenilir hale getirir.

### Bağlantı Durumu Hook'u (Connection Status Hook) Eklendi
**Katkıda bulunan:** @nexxai

Bir bağlantı durumu hook'u, kullanıcı deneyimi (UX) için büyük bir kazanımdır: "bağlı / yeniden bağlanıyor / çevrimdışı" durumlarını net bir şekilde yansıtabilir ve bunları UI'ınızda zarifçe ele alabilirsiniz. Güncellemelerin gerçekten canlı olup olmadığını bilmenin kullanıcı güvenine bağlı olduğu, gerçek zamanlı ağırlıklı uygulamalar için harikadır.

### Olay Aşırı Yüklemesi (Events Overloading)
**Katkıda bulunan:** @joetannenbaum

Olay aşırı yüklemesi, olayları nasıl eşleyebileceğinizi (map) ve işleyebileceğinizi genişleterek, geçici çözüm kodlarına (workaround code) ihtiyaç duymadan gelişmiş gerçek zamanlı kalıpları ifade etmeyi kolaylaştırır. Birçok olay türünü daha temiz, daha sürdürülebilir bir istemci tarafı katmanında birleştirirken kullanışlıdır.

## Horizon

### `horizon:listen` Komutu Eklendi
**Katkıda bulunan:** @mathiasgrimm

Geliştirme sırasında dosya değişikliklerini izleyen ve Horizon'u otomatik olarak yeniden başlatan yeni bir `horizon:listen` komutu tanıtıldı. Bu yeni komut, Laravel'in yerleşik `queue:listen` komutunun davranışını yansıtır.

## Pint

### `agent` Formatı Eklendi
**Katkıda bulunan:** @nunomaduro

Pint, OpenCode veya Claude Code içinde yürütüldüğünde otomatik olarak kullanılan yeni bir `--format agent` bayrağı eklendi.

Varsayılan Pint çıktısı insanlar için tasarlanmıştır: renkli, ilerleme noktaları ve biçimlendirilmiş özetler içerir. Yapay zeka ajanlarının farklı bir şeye ihtiyacı vardır: güvenilir bir şekilde ayrıştırabilecekleri yapılandırılmış JSON, biçimlendirilmiş metinden çıkarım yapmak yerine "status":"pass" gibi belirsiz olmayan durum değerleri, bağlam penceresinden (context window) tasarruf sağlayan ve token maliyetini azaltan minimum çıktı ve ANSI renkleri veya biçimlendirme gürültüsü olmadan deterministik sonuçlar.

## Prompts

### Grid Bileşeni (Grid Component) Eklendi
**Katkıda bulunan:** @pushpak1300

Prompts'teki yeni `grid` bileşeni, geliştiricilerin kolayca duyarlı (responsive) grid tabanlı düzenler oluşturmasına ve verileri net bir şekilde sunmasına olanak tanır.

## Pulse

### Livewire 4 Desteği Eklendi
**Katkıda bulunan:** @joshhanley

Pulse, Livewire 4 için hazır. Bu, uygulama gözlemlenebilirlik (observability) UI'ınızı ilerlerken uyumlu tutar. Bu, yığınınızı modernleştirirken daha az engel anlamına gelir.

## VS Code Eklentisi

### Livewire 4 Desteği
**Katkıda bulunan:** @TitasGailius

Eklentide birinci sınıf Livewire 4 desteği, bileşenler oluştururken daha iyi düzenleyici anlayışıyla akışta kalmanıza yardımcı olur: daha az bağlam değiştirme, daha fazla ürün teslimi.

### Docker Desteği
**Katkıda bulunan:** @TitasGailius

Geliştirilmiş Docker desteği, konteynerler içinde Laravel geliştirmeyi VS Code'da daha doğal hissettirir. Bu, ekiplerin ortamları standartlaştırmasına ve "benim makinemde çalışıyor" (works on my machine) sürtüşmesini azaltmasına yardımcı olur.

### Kapsamlar (Scopes) için Uygulamaya (Implementation) Giden Markdown Bağlantıları Eklendi
**Katkıda bulunan:** @N1ebieski

Bağlantılar, sorgu kapsamlarını (query scopes) gezilebilir bir deneyime dönüştürür. Kullanımdan uygulamaya daha hızlı atlar. Bu, özellikle çok sayıda paylaşılan kapsamın bulunduğu büyük kod tabanlarında değerlidir.

### Laravel Nitelikleri (Attributes) için Destek
**Katkıda bulunan:** @N1ebieski

Daha iyi nitelik desteği, düzenleyicinin modern Laravel kalıplarını anlamasına yardımcı olur, gezinmeyi iyileştirir ve yanlış pozitifleri (false positives) azaltır. Projeniz PHP niteliklerine dayanıyorsa, bu, günlük düzenleme işlerini gözle görülür şekilde daha akıcı hale getirir.

### Artisan `make` Komutlarının VS Code Gezgini (Explorer) ile Entegrasyonu
**Katkıda bulunan:** @N1ebieski

Zaten çalıştığınız yerde dosyalar oluşturun: `artisan make:*` komutlarını gezgine ve bağlam menüsüne entegre etmek, terminale geçişi azaltır ve iskeleyi (scaffolding) proje yapınıza yakın tutar.

### `FormRequest`'te Otomatik Tamamlama Kuralları (Autocompletion Rules) için Destek
**Katkıda bulunan:** @N1ebieski

`FormRequest` doğrulama kuralları için daha akıllı otomatik tamamlama, daha az yazım hatası ve daha hızlı doğrulama yazımı anlamına gelir. Bu, özellikle karmaşık kural kümeleriyle uğraşırken faydalıdır.

### Depoya (Repository) ve Docblock Oluşturucuya (Docblock Generator) Kapsam Parametreleri (Scope Parameters) Eklendi
**Katkıda bulunan:** @N1ebieski

Kapsam parametrelerini içeren docblock'lar ve depo ipuçları, otomatik tamamlama doğruluğunu artırır ve ekip arkadaşları için "keşfedilebilirliği" iyileştirir. Böylece mevcut kapsamları kullanmak, metod imzasını okumak kadar kolay hale gelir.

## Aralık 2025

## Laravel Framework 12.x

### [12.x] Bir HTTP Yanıtı Oluşturulduktan Sonra Geri Çağrıları (Callbacks) Çalıştırma Yeteneği Eklendi
**Katkıda bulunan:** @cosmastech

HTTP İstemcisine (HTTP Client) yeni bir `afterResponse` metodu eklendi. Bu, kullanıcının bir HTTP çağrısından döndürülen yanıtı incelemesine ve değiştirmesine olanak tanır:
```php
Http::acceptJson()
    ->withHeader('X-Shopify-Access-Token', $credentials->token)
    ->baseUrl("https://{$credentials->shop_domain}.myshopify.com/admin/api/2025-10/")
    ->afterResponse(
        // Başlıktaki tüm kullanımdan kaldırma (deprecation) bildirimlerini raporla
        function (Response $response) use ($credentials) {
            if ($header = $response->header('X-Shopify-API-Deprecated-Reason')) {
                event(new ShopifyDeprecationNotice($credentials->shop, $header));
            }
        })
    ->afterResponse(
        // Yanıtı kendi özel yanıt sınıfımıza dönüştür
        fn (Response $response) => new ShopifyResponse($response->toPsrResponse()),
    )
    ->afterResponse(
        // Sorgunun maliyetini raporla
        static fn (ShopifyResponse $response) => QueryCostResponse::report(
            $response->getQueryCost(),
            $credentials->shop
        ),
    );
```

### [12.x] Havuzda (Pool) Daha Temiz Zincirleme (Chaining) için `FluentPromise` Tanıtıldı
**Katkıda bulunan:** @cosmastech

`FluentPromise`'in tanıtılmasıyla `Http::pool`'da kullanıcı alanında (userland) zincirleme artık mümkün. Daha önce zincir, orijinal yanıtı döndürerek esnekliği sınırlandırıyordu. Bu değişiklikle, kullanıcılar temeldeki diziye erişebilir ve istediklerini döndürebilir:
```php
$response = Http::pool(function (Pool $pool) {
    $pool->as('cosmatech')
        ->get('https://cosmastech.com')
        ->then(
            fn ($response) => strlen($response->body())
        );
    $pool->as('laravel')
        ->get('https://laravel.com')
        ->then(
            fn ($response) => strlen($response->body())
        );
});

/*
array:2 [
  "cosmatech" => 9095
  "laravel" => 1422023
]
*/
```

### `lazy` Nesne ve `proxy` Nesne Destek Yardımcıları Tanıtıldı
**Katkıda bulunan:** @timacdonald

PHP'nin yeni `lazy` nesnelerini kullanmayı daha ergonomik hale getirmek için iki yeni yardımcı olan `lazy` ve `proxy` eklendi.
```php
<?php

use Illuminate\Support\Facades\Http;
use Vendor\Facades\ResultFactory;
use Vendor\Result;
use function Illuminate\Support\lazy;
use function Illuminate\Support\proxy;

$response = Http::get(...);

$instance = lazy(Result::class, fn () => [$response->json()]);
$instance = proxy(Result::class, fn () => $response->json());
```