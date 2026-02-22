# Laravel 12 Dokümantasyonu: Varlık Paketleme (Vite)

## Giriş

Vite, son derece hızlı bir geliştirme ortamı sağlayan ve kodunuzu üretim için paketleyen (bundle) modern bir ön yüz derleme (frontend build) aracıdır. Laravel ile uygulama oluştururken, tipik olarak Vite'i uygulamanızın CSS ve JavaScript dosyalarını üretime hazır varlıklar (production-ready assets) halinde paketlemek için kullanacaksınız.

Laravel, resmi bir eklenti (plugin) ve geliştirme ve üretim için varlıklarınızı yüklemek üzere bir Blade direktifi sağlayarak Vite ile sorunsuz bir şekilde entegre olur.

## Kurulum ve Yapılandırma (Installation & Setup)

Aşağıdaki dokümantasyon, Laravel Vite eklentisinin manuel olarak nasıl kurulacağını ve yapılandırılacağını açıklar. Ancak, Laravel'in başlangıç kitleri (starter kits) zaten tüm bu iskeleyi (scaffolding) içerir ve Laravel ve Vite ile başlamanın en hızlı yoludur.

### Node Kurulumu (Installing Node)
Vite ve Laravel eklentisini çalıştırmadan önce Node.js (16+) ve NPM'nin kurulu olduğundan emin olmalısınız:
```bash
node -v
npm -v
```
Node'un resmi web sitesindeki basit grafiksel kurucuları kullanarak Node ve NPM'nin en son sürümünü kolayca kurabilirsiniz. Veya Laravel Sail kullanıyorsanız, Node ve NPM'yi Sail aracılığıyla çağırabilirsiniz:
```bash
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

### Vite ve Laravel Eklentisini Kurma (Installing Vite and the Laravel Plugin)
Yeni bir Laravel kurulumunda, uygulamanızın dizin yapısının kökünde bir `package.json` dosyası bulacaksınız. Varsayılan `package.json` dosyası, Vite ve Laravel eklentisini kullanmaya başlamak için ihtiyacınız olan her şeyi zaten içerir. Uygulamanızın ön yüz (frontend) bağımlılıklarını NPM aracılığıyla kurabilirsiniz:
```bash
npm install
```

### Vite'i Yapılandırma (Configuring Vite)
Vite, projenizin kökündeki `vite.config.js` dosyası aracılığıyla yapılandırılır. Bu dosyayı ihtiyaçlarınıza göre özelleştirmekte özgürsünüz ve uygulamanızın gerektirdiği `@vitejs/plugin-react`, `@sveltejs/vite-plugin-svelte` veya `@vitejs/plugin-vue` gibi diğer eklentileri de kurabilirsiniz.

Laravel Vite eklentisi, uygulamanız için giriş noktalarını (entry points) belirtmenizi gerektirir. Bunlar JavaScript veya CSS dosyaları olabilir ve TypeScript, JSX, TSX ve Sass gibi ön işlemden geçirilmiş (preprocessed) dilleri içerebilir.
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```
Inertia kullanılarak oluşturulan uygulamalar da dahil olmak üzere bir Tek Sayfa Uygulaması (SPA) oluşturuyorsanız, Vite CSS giriş noktaları olmadan en iyi şekilde çalışır. Bunun yerine, CSS'inizi JavaScript aracılığıyla içe aktarmalısınız (import). Tipik olarak bu, uygulamanızın `resources/js/app.js` dosyasında yapılır:
```javascript
import './bootstrap';
import '../css/app.css'; // CSS burada içe aktarılır
```
Laravel eklentisi ayrıca birden çok giriş noktasını ve SSR giriş noktaları gibi gelişmiş yapılandırma seçeneklerini de destekler.

#### Güvenli Bir Geliştirme Sunucusuyla Çalışma (Working With a Secure Development Server)
Yerel geliştirme web sunucunuz uygulamanıza HTTPS üzerinden hizmet veriyorsa, Vite geliştirme sunucusuna bağlanırken sorunlarla karşılaşabilirsiniz.

Laravel Herd kullanıyorsanız ve siteyi güvenceye aldıysanız (secured) veya Laravel Valet kullanıyorsanız ve uygulamanıza karşı `secure` komutunu çalıştırdıysanız, Laravel Vite eklentisi otomatik olarak oluşturulan TLS sertifikasını algılayacak ve sizin için kullanacaktır.

Siteyi uygulamanın dizin adıyla eşleşmeyen bir ana bilgisayar (host) kullanarak güvenceye aldıysanız, ana bilgisayarı uygulamanızın `vite.config.js` dosyasında manuel olarak belirtebilirsiniz:
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test',
        }),
    ],
});
```
Başka bir web sunucusu kullanırken, güvenilir bir sertifika oluşturmalı ve Vite'i oluşturulan sertifikaları kullanacak şekilde manuel olarak yapılandırmalısınız:
```javascript
// ...
import fs from 'fs'; // Dosya sistemi modülünü içe aktar

const host = 'my-app.test'; // Ana bilgisayar adını tanımla

export default defineConfig({
    // ...
    server: {
        host,
        hmr: { host },
        https: {
            key: fs.readFileSync(`/path/to/${host}.key`),
            cert: fs.readFileSync(`/path/to/${host}.crt`),
        },
    },
});
```
Sisteminiz için güvenilir bir sertifika oluşturamıyorsanız, `@vitejs/plugin-basic-ssl` eklentisini kurabilir ve yapılandırabilirsiniz. Güvenilmeyen sertifikalar kullanırken, `npm run dev` komutunu çalıştırdığınızda konsoldaki "Local" bağlantısını izleyerek tarayıcınızda Vite'in geliştirme sunucusu için sertifika uyarısını kabul etmeniz gerekecektir.

#### Geliştirme Sunucusunu WSL2 Üzerinde Sail'de Çalıştırma (Running the Development Server in Sail on WSL2)
Windows Subsystem for Linux 2 (WSL2) üzerinde Laravel Sail içinde Vite geliştirme sunucusunu çalıştırırken, tarayıcının geliştirme sunucusuyla iletişim kurabildiğinden emin olmak için `vite.config.js` dosyanıza aşağıdaki yapılandırmayı eklemelisiniz:
```javascript
// ...

export default defineConfig({
    // ...
    server: {
        hmr: {
            host: 'localhost',
        },
    },
});
```
Geliştirme sunucusu çalışırken dosya değişiklikleriniz tarayıcıya yansımıyorsa, Vite'in `server.watch.usePolling` seçeneğini de yapılandırmanız gerekebilir.

### Betiklerinizi (Scripts) ve Stillerinizi (Styles) Yükleme (Loading Your Scripts and Styles)
Vite giriş noktalarınızı yapılandırdıktan sonra, artık bunları uygulamanızın kök şablonunun (root template) `<head>` bölümüne eklediğiniz bir `@vite()` Blade direktifinde referans gösterebilirsiniz:
```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```
CSS'inizi JavaScript aracılığıyla içe aktarıyorsanız, yalnızca JavaScript giriş noktasını eklemeniz gerekir:
```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```
`@vite` direktifi otomatik olarak Vite geliştirme sunucusunu algılayacak ve Sıcak Modül Değişimini (Hot Module Replacement - HMR) etkinleştirmek için Vite istemcisini enjekte edecektir. Derleme (build) modunda, direktif, içe aktarılan tüm CSS dahil olmak üzere derlenmiş ve sürümlenmiş (versioned) varlıklarınızı yükleyecektir.

Gerekirse, `@vite` direktifini çağırırken derlenmiş varlıklarınızın derleme yolunu (build path) da belirtebilirsiniz:
```blade
<!doctype html>
<head>
    {{-- Verilen derleme yolu, public yoluna göredir (relative). --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

#### Satır İçi (Inline) Varlıklar (Inline Assets)
Bazen varlıkların sürümlenmiş URL'sine bağlantı vermek yerine, varlıkların ham içeriğini (raw content) dahil etmek gerekebilir. Örneğin, bir PDF oluşturucuya HTML içeriği aktarırken varlık içeriğini doğrudan sayfanıza dahil etmeniz gerekebilir. Vite facade'ı tarafından sağlanan `content` metodunu kullanarak Vite varlıklarının içeriğini çıktı olarak verebilirsiniz:
```blade
@use('Illuminate\Support\Facades\Vite')

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

## Vite'i Çalıştırma (Running Vite)

Vite'i çalıştırmanın iki yolu vardır. Yerel olarak geliştirme yaparken kullanışlı olan `dev` komutu aracılığıyla geliştirme sunucusunu çalıştırabilirsiniz. Geliştirme sunucusu, dosyalarınızdaki değişiklikleri otomatik olarak algılayacak ve bunları açık olan herhangi bir tarayıcı penceresinde anında yansıtacaktır.

Veya, `build` komutunu çalıştırmak, uygulamanızın varlıklarını sürümleyecek ve paketleyerek üretime dağıtmaya hazır hale getirecektir:
```bash
# Vite geliştirme sunucusunu çalıştır...
npm run dev

# Varlıkları üretim için derle ve sürümle...
npm run build
```
Geliştirme sunucusunu WSL2 üzerinde Sail'de çalıştırıyorsanız, bazı ek yapılandırma seçeneklerine ihtiyacınız olabilir.

## JavaScript ile Çalışma (Working With JavaScript)

### Takma Adlar (Aliases)
Varsayılan olarak, Laravel eklentisi, hızlı bir başlangıç yapmanıza ve uygulamanızın varlıklarını kolayca içe aktarmanıza yardımcı olacak yaygın bir takma ad (alias) sağlar:
```javascript
{
    '@' => '/resources/js'
}
```
`'@'` takma adını, `vite.config.js` yapılandırma dosyasına kendi takma adınızı ekleyerek geçersiz kılabilirsiniz (overwrite):
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

### Vue ile Kullanım
Ön yüzünüzü Vue framework'ünü kullanarak oluşturmak isterseniz, `@vitejs/plugin-vue` eklentisini de kurmanız gerekecektir:
```bash
npm install --save-dev @vitejs/plugin-vue
```
Ardından eklentiyi `vite.config.js` yapılandırma dosyanıza dahil edebilirsiniz. Vue eklentisini Laravel ile kullanırken ihtiyaç duyacağınız birkaç ek seçenek vardır:
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // Vue eklentisi, Tek Dosya Bileşenlerinde (Single File Components) referans verilen
                    // varlık URL'lerini Laravel web sunucusunu işaret edecek şekilde yeniden yazar.
                    // Bunu `null` olarak ayarlamak, Laravel eklentisinin bunun yerine
                    // varlık URL'lerini Vite sunucusunu işaret edecek şekilde yeniden yazmasına izin verir.
                    base: null,

                    // Vue eklentisi mutlak URL'leri ayrıştıracak ve bunları diskteki dosyalara
                    // giden mutlak yollar olarak ele alacaktır. Bunu `false` olarak ayarlamak,
                    // mutlak URL'leri olduğu gibi bırakır, böylece beklendiği gibi public
                    // dizinindeki varlıklara referans verebilirler.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```
Laravel'in başlangıç kitleri (starter kits) zaten uygun Laravel, Vue ve Vite yapılandırmasını içerir. Bu başlangıç kitleri, Laravel, Vue ve Vite ile başlamanın en hızlı yolunu sunar.

### React ile Kullanım
Ön yüzünüzü React framework'ünü kullanarak oluşturmak isterseniz, `@vitejs/plugin-react` eklentisini de kurmanız gerekecektir:
```bash
npm install --save-dev @vitejs/plugin-react
```
Ardından eklentiyi `vite.config.js` yapılandırma dosyanıza dahil edebilirsiniz:
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```
JSX içeren tüm dosyaların bir `.jsx` veya `.tsx` uzantısına sahip olduğundan emin olmanız ve gerekirse yukarıda gösterildiği gibi giriş noktanızı (entry point) güncellemeyi unutmamanız gerekecektir.

Ayrıca, mevcut `@vite` direktifinizin yanına ek `@viteReactRefresh` Blade direktifini de eklemeniz gerekecektir.
```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```
`@viteReactRefresh` direktifi, `@vite` direktifinden önce çağrılmalıdır.

Laravel'in başlangıç kitleri zaten uygun Laravel, React ve Vite yapılandırmasını içerir. Bu başlangıç kitleri, Laravel, React ve Vite ile başlamanın en hızlı yolunu sunar.

### Svelte ile Kullanım
Ön yüzünüzü Svelte framework'ünü kullanarak oluşturmak isterseniz, `@sveltejs/vite-plugin-svelte` eklentisini de kurmanız gerekecektir:
```bash
npm install --save-dev @sveltejs/vite-plugin-svelte
```
Ardından eklentiyi `vite.config.js` yapılandırma dosyanıza dahil edebilirsiniz.
```javascript
import { svelte } from '@sveltejs/vite-plugin-svelte';
import laravel from 'laravel-vite-plugin';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/js/app.ts'],
      ssr: 'resources/js/ssr.ts',
      refresh: true,
    }),
    svelte(),
  ],
});
```
Laravel'in başlangıç kitleri zaten uygun Laravel, Svelte ve Vite yapılandırmasını içerir. Bu başlangıç kitleri, Laravel, Svelte ve Vite ile başlamanın en hızlı yolunu sunar.

### Inertia ile Kullanım
Laravel Vite eklentisi, Inertia sayfa bileşenlerinizi çözümlemenize (resolve) yardımcı olacak kullanışlı bir `resolvePageComponent` fonksiyonu sağlar. Aşağıda, yardımcının Vue 3 ile kullanımına bir örnek verilmiştir; ancak, fonksiyonu React veya Svelte gibi diğer framework'lerde de kullanabilirsiniz:
```javascript
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```
Inertia ile Vite'in kod bölme (code splitting) özelliğini kullanıyorsanız, varlık önceden getirmeyi (asset prefetching) yapılandırmanızı öneririz.

Laravel'in başlangıç kitleri zaten uygun Laravel, Inertia ve Vite yapılandırmasını içerir. Bu başlangıç kitleri, Laravel, Inertia ve Vite ile başlamanın en hızlı yolunu sunar.

### URL İşleme (URL Processing)
Vite'i kullanırken ve uygulamanızın HTML, CSS veya JS'sinde varlıklara referans verirken göz önünde bulundurmanız gereken birkaç nokta vardır. İlk olarak, varlıklara mutlak bir yolla (absolute path) referans verirseniz, Vite varlığı derlemeye dahil etmeyecektir; bu nedenle, varlığın `public` dizininizde mevcut olduğundan emin olmalısınız. Özel bir CSS giriş noktası kullanırken mutlak yolları kullanmaktan kaçınmalısınız çünkü geliştirme sırasında tarayıcılar bu yolları `public` dizininizden değil, CSS'in barındırıldığı Vite geliştirme sunucusundan yüklemeye çalışacaktır.

Göreceli (relative) varlık yollarına referans verirken, yolların referans verildikleri dosyaya göre olduğunu unutmamalısınız. Göreceli bir yol aracılığıyla referans verilen tüm varlıklar, Vite tarafından yeniden yazılacak, sürümlenecek ve paketlenecektir.

Aşağıdaki proje yapısını düşünün:
```
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```
Aşağıdaki örnek, Vite'in göreceli ve mutlak URL'leri nasıl ele alacağını göstermektedir:
```html
<!-- Bu varlık Vite tarafından işlenmez ve derlemeye dahil edilmez -->
<img src="/taylor.png">

<!-- Bu varlık Vite tarafından yeniden yazılacak, sürümlenecek ve paketlenecek -->
<img src="../../images/abigail.png">
```

## Stil Sayfaları (Stylesheets) ile Çalışma

Laravel'in başlangıç kitleri zaten uygun Tailwind ve Vite yapılandırmasını içerir. Veya başlangıç kitlerimizden birini kullanmadan Laravel ile Tailwind kullanmak isterseniz, Tailwind'in kurulum kılavuzuna göz atın.