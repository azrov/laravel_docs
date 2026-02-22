# Laravel 12 Dokümantasyonu: Ön Yüz (Frontend)

## Giriş

Laravel, yönlendirme (routing), doğrulama (validation), önbellekleme (caching), kuyruklar (queues), dosya depolama ve daha fazlası gibi modern web uygulamaları oluşturmak için ihtiyaç duyduğunuz tüm özellikleri sağlayan bir arka uç (backend) framework'üdür. Ancak, geliştiricilere, uygulamalarının ön yüzünü (frontend) oluşturmak için güçlü yaklaşımlar da dahil olmak üzere, güzel bir tam yığın (full-stack) deneyimi sunmanın önemli olduğuna inanıyoruz.

Laravel ile bir uygulama oluştururken ön yüz geliştirmeyi ele almanın iki ana yolu vardır ve hangi yaklaşımı seçeceğiniz, ön yüzünüzü PHP kullanarak mı yoksa React, Vue veya Svelte gibi JavaScript framework'leri kullanarak mı oluşturmak istediğinize göre belirlenir. Uygulamanız için en iyi ön yüz geliştirme yaklaşımı konusunda bilinçli bir karar verebilmeniz için bu seçeneklerin her ikisini de aşağıda tartışacağız.

## PHP Kullanmak

### PHP ve Blade

Geçmişte, çoğu PHP uygulaması, istek sırasında bir veritabanından alınan verileri görüntüleyen PHP `echo` ifadeleriyle serpiştirilmiş basit HTML şablonları kullanarak tarayıcıya HTML render ederdi:
```php
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```
Laravel'de, HTML render etmeye yönelik bu yaklaşım, görünümler (views) ve Blade kullanılarak hala gerçekleştirilebilir. Blade, verileri görüntülemek, veriler üzerinde yineleme yapmak ve daha fazlası için kullanışlı, kısa bir sözdizimi sağlayan son derece hafif bir şablonlama dilidir (templating language):
```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```
Uygulamaları bu şekilde oluştururken, form gönderimleri ve diğer sayfa etkileşimleri tipik olarak sunucudan tamamen yeni bir HTML belgesi alır ve tüm sayfa tarayıcı tarafından yeniden render edilir. Bugün bile, birçok uygulama, ön yüzlerinin basit Blade şablonları kullanılarak bu şekilde oluşturulmasına mükemmel şekilde uygun olabilir.

#### Artan Beklentiler

Ancak, web uygulamalarına ilişkin kullanıcı beklentileri olgunlaştıkça, birçok geliştirici daha dinamik, daha cilalı hissettiren etkileşimlere sahip ön yüzler oluşturma ihtiyacı duymuştur. Bu nedenle, bazı geliştiriciler uygulamalarının ön yüzünü React, Vue veya Svelte gibi JavaScript framework'leri kullanarak oluşturmaya başlamayı tercih etmektedir.

Diğerleri ise rahat oldukları arka uç diline bağlı kalmayı tercih ederek, öncelikle tercih ettikleri arka uç dilini kullanırken modern web uygulaması kullanıcı arayüzleri (UI) oluşturmaya izin veren çözümler geliştirmiştir. Örneğin, Rails ekosisteminde bu, Turbo Hotwire ve Stimulus gibi kütüphanelerin oluşturulmasını teşvik etmiştir.

Laravel ekosisteminde ise, öncelikle PHP kullanarak modern, dinamik ön yüzler oluşturma ihtiyacı, Laravel Livewire ve Alpine.js'in oluşturulmasına yol açmıştır.

### Livewire

Laravel Livewire, React, Vue veya Svelte gibi modern JavaScript framework'leriyle oluşturulmuş ön yüzler kadar dinamik, modern ve canlı hissedilen, Laravel destekli ön yüzler oluşturmak için bir framework'tür.

Livewire kullanırken, kullanıcı arayüzünüzün ayrı bir bölümünü render eden ve uygulamanızın ön yüzünden çağrılıp etkileşim kurulabilecek metotlar ve veriler sunan Livewire "bileşenleri" (components) oluşturacaksınız. Örneğin, basit bir "Sayaç" (Counter) bileşeni şöyle görünebilir:
```php
<?php

use Livewire\Component;

new class extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }
};
?>

<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```
Gördüğünüz gibi Livewire, Laravel uygulamanızın ön yüzünü ve arka yüzünü birbirine bağlayan `wire:click` gibi yeni HTML nitelikleri (attributes) yazmanıza olanak tanır. Ayrıca, basit Blade ifadelerini kullanarak bileşeninizin mevcut durumunu render edebilirsiniz.

Birçok kişi için Livewire, Laravel ile ön yüz geliştirmede devrim yaratarak, modern, dinamik web uygulamaları oluştururken Laravel'in rahatlığı içinde kalmalarına olanak tanımıştır. Tipik olarak, Livewire kullanan geliştiriciler, yalnızca ihtiyaç duyulan yerde (örneğin bir diyalog penceresi render etmek için) JavaScript'i ön yüzlerine "serpiştirmek" (sprinkle) amacıyla Alpine.js'i de kullanacaklardır.

Laravel'de yeniyseniz, görünümler (views) ve Blade'in temel kullanımına aşina olmanızı öneririz. Ardından, etkileşimli Livewire bileşenleriyle uygulamanızı bir sonraki seviyeye nasıl taşıyacağınızı öğrenmek için resmi Laravel Livewire dokümantasyonuna bakın.

### Başlangıç Kitleri (Starter Kits)

Ön yüzünüzü PHP ve Livewire kullanarak oluşturmak isterseniz, uygulamanızın geliştirilmesine hızlı bir başlangıç yapmak için Livewire başlangıç kitimizden yararlanabilirsiniz.

## React, Vue veya Svelte Kullanmak

Laravel ve Livewire kullanarak modern ön yüzler oluşturmak mümkün olsa da, birçok geliştirici hala React, Vue veya Svelte gibi bir JavaScript framework'ünün gücünden yararlanmayı tercih etmektedir. Bu, geliştiricilerin NPM aracılığıyla sunulan zengin JavaScript paketleri ve araçları ekosisteminden avantaj sağlamasına olanak tanır.

Ancak, ek araçlar olmadan, Laravel'i React, Vue veya Svelte ile eşleştirmek, istemci tarafı yönlendirme (client-side routing), veri hidrasyonu (data hydration) ve kimlik doğrulama (authentication) gibi çeşitli karmaşık sorunları çözme ihtiyacıyla bizi baş başa bırakır. İstemci tarafı yönlendirme genellikle Next ve Nuxt gibi görüşe dayalı (opinionated) React / Vue framework'leri kullanılarak basitleştirilir; ancak, Laravel gibi bir arka uç framework'ünü bu ön yüz framework'leriyle eşleştirirken veri hidrasyonu ve kimlik doğrulama, çözülmesi karmaşık ve zahmetli sorunlar olmaya devam etmektedir.

Ayrıca, geliştiriciler genellikle her iki depo (repository) arasında bakım, sürüm ve dağıtımları koordine etmek zorunda kalarak iki ayrı kod deposunu yönetmek durumunda kalırlar. Bu sorunlar aşılamaz olmasa da, uygulama geliştirmenin üretken veya keyifli bir yolu olduğuna inanmıyoruz.

### Inertia

Neyse ki Laravel, her iki dünyanın da en iyisini sunar. Inertia, Laravel uygulamanız ile modern React, Vue veya Svelte ön yüzünüz arasındaki boşluğu doldurarak, yönlendirme (routing), veri hidrasyonu (data hydration) ve kimlik doğrulama (authentication) için Laravel rotalarını (routes) ve controller'larını kullanırken React, Vue veya Svelte kullanarak tam teşekküllü, modern ön yüzler oluşturmanıza olanak tanır - tüm bunlar tek bir kod deposu içinde. Bu yaklaşımla, her iki aracın da yeteneklerini köreltmeden hem Laravel'in hem de React / Vue / Svelte'in tüm gücünün keyfini çıkarabilirsiniz.

Inertia'yı Laravel uygulamanıza yükledikten sonra, rotaları ve controller'ları normal bir şekilde yazacaksınız. Ancak, controller'ınızdan bir Blade şablonu döndürmek yerine, bir Inertia sayfası döndüreceksiniz:
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Belirli bir kullanıcının profilini göster.
     */
    public function show(string $id): Response
    {
        return Inertia::render('users/show', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```
Bir Inertia sayfası, tipik olarak uygulamanızın `resources/js/pages` dizininde depolanan bir React, Vue veya Svelte bileşenine karşılık gelir. `Inertia::render` metodu aracılığıyla sayfaya verilen veriler, sayfa bileşeninin "prop"larını (property/özellik) doldurmak (hydrate) için kullanılacaktır:
```jsx
import Layout from '@/layouts/authenticated';
import { Head } from '@inertiajs/react';

export default function Show({ user }) {
    return (
        <Layout>
            <Head title="Welcome" />
            <h1>Welcome</h1>
            <p>Hello {user.name}, welcome to Inertia.</p>
        </Layout>
    )
}
```
Gördüğünüz gibi Inertia, ön yüzünüzü oluştururken React, Vue veya Svelte'in tüm gücünden yararlanmanıza olanak tanırken, Laravel destekli arka uç ile JavaScript destekli ön yüz arasında hafif bir köprü sağlar.

#### Sunucu Taraflı Render Etme (Server-Side Rendering)

Uygulamanız sunucu taraflı render etme (SSR) gerektirdiği için Inertia'ya dalmak konusunda endişeliyseniz, endişelenmeyin. Inertia, sunucu taraflı render etme desteği sunar. Ve uygulamanızı Laravel Cloud veya Laravel Forge aracılığıyla dağıtırken, Inertia'nın sunucu taraflı render etme sürecinin her zaman çalışır durumda olmasını sağlamak çok kolaydır.

### Başlangıç Kitleri (Starter Kits)

Ön yüzünüzü Inertia ve React / Vue / Svelte kullanarak oluşturmak isterseniz, uygulamanızın geliştirilmesine hızlı bir başlangıç yapmak için React, Vue veya Svelte uygulama başlangıç kitlerimizden yararlanabilirsiniz. Bu başlangıç kitlerinin her biri, bir sonraki büyük fikrinizi oluşturmaya başlayabilmeniz için uygulamanızın arka uç ve ön uç kimlik doğrulama akışını (authentication flow) Inertia, React / Vue / Svelte, Tailwind ve Vite kullanarak hazır hale getirir (scaffold).

## Varlıkları Paketleme (Bundling Assets)

Ön yüzünüzü Blade ve Livewire veya React / Vue / Svelte ve Inertia kullanarak geliştirmeyi seçseniz de, uygulamanızın CSS'ini üretime hazır varlıklar halinde paketlemeniz (bundle) muhtemelen gerekecektir. Elbette, uygulamanızın ön yüzünü React, Vue veya Svelte ile oluşturmayı seçerseniz, bileşenlerinizi tarayıcı tarafından okunabilir JavaScript varlıkları halinde paketlemeniz de gerekecektir.

Varsayılan olarak Laravel, varlıklarınızı paketlemek için Vite'i kullanır. Vite, yerel geliştirme sırasında yıldırım hızında derleme süreleri (build times) ve neredeyse anında Sıcak Modül Değişimi (Hot Module Replacement - HMR) sağlar. Başlangıç kitlerimizi kullananlar da dahil olmak üzere tüm yeni Laravel uygulamalarında, Vite'i Laravel uygulamalarıyla kullanmayı keyifli hale getiren hafif Laravel Vite eklentimizi yükleyen bir `vite.config.js` dosyası bulacaksınız.

Laravel ve Vite ile başlamanın en hızlı yolu, ön yüz ve arka uç kimlik doğrulama iskeleti (scaffolding) sağlayarak uygulamanıza hızlı bir başlangıç yaptıran uygulama başlangıç kitlerimizi kullanmaktır.

Vite'i Laravel ile kullanma hakkında daha detaylı dokümantasyon için lütfen varlıklarınızı paketleme ve derleme konusundaki özel dokümantasyonumuza bakın.