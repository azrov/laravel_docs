# Laravel 12 Dokümantasyonu: Başlangıç Kitleri (Starter Kits)

## Giriş

Yeni Laravel uygulamanızı oluştururken size hızlı bir başlangıç sağlamak için uygulama başlangıç kitleri sunmaktan mutluluk duyuyoruz. Bu başlangıç kitleri, bir sonraki Laravel uygulamanızı oluştururken size bir ön hazırlık sağlar ve uygulamanızın kullanıcılarını kaydetmek ve kimlik doğrulamak için ihtiyaç duyduğunuz rotaları (routes), controller'ları ve görünümleri (views) içerir. Başlangıç kitleri, kimlik doğrulama (authentication) sağlamak için Laravel Fortify'i kullanır.

Bu başlangıç kitlerini kullanmakta özgür olsanız da, bunlar zorunlu değildir. Sadece yeni bir Laravel kopyası kurarak uygulamanızı sıfırdan oluşturmakta özgürsünüz. Her iki durumda da, harika bir şey yapacağınızı biliyoruz!

## Bir Başlangıç Kiti Kullanarak Uygulama Oluşturma

Başlangıç kitlerimizden birini kullanarak yeni bir Laravel uygulaması oluşturmak için öncelikle PHP ve Laravel CLI aracını kurmalısınız. Eğer zaten PHP ve Composer'ınız yüklüyse, Laravel kurucusu CLI aracını Composer aracılığıyla kurabilirsiniz:
```bash
composer global require laravel/installer
```
Ardından, Laravel kurucusu CLI'ını kullanarak yeni bir Laravel uygulaması oluşturun. Laravel kurucusu, tercih ettiğiniz başlangıç kitini seçmeniz için size soracaktır:
```bash
laravel new my-app
```
Laravel uygulamanızı oluşturduktan sonra, yalnızca NPM aracılığıyla ön yüz (frontend) bağımlılıklarını kurmanız ve Laravel geliştirme sunucusunu başlatmanız gerekir:
```bash
cd my-app
npm install && npm run build
composer run dev
```
Laravel geliştirme sunucusunu başlattıktan sonra, uygulamanıza web tarayıcınızdan `http://localhost:8000` adresinden erişebileceksiniz.

## Mevcut Başlangıç Kitleri

### React Başlangıç Kiti
React başlangıç kitimiz, Inertia kullanarak bir React ön yüzü ile Laravel uygulamaları oluşturmak için sağlam, modern bir başlangıç noktası sağlar.

Inertia, klasik sunucu tarafı yönlendirme (server-side routing) ve controller'ları kullanarak modern, tek sayfa React uygulamaları (SPA) oluşturmanıza olanak tanır. Bu, React'in ön yüz gücü ile Laravel'in inanılmaz arka uç üretkenliğini ve yıldırım hızındaki Vite derlemesini bir arada kullanmanızı sağlar.

React başlangıç kiti, React 19, TypeScript, Tailwind ve shadcn/ui bileşen kütüphanesini kullanır.

### Svelte Başlangıç Kiti
Svelte başlangıç kitimiz, Inertia kullanarak bir Svelte ön yüzü ile Laravel uygulamaları oluşturmak için sağlam, modern bir başlangıç noktası sağlar.

Inertia, klasik sunucu tarafı yönlendirme ve controller'ları kullanarak modern, tek sayfa Svelte uygulamaları oluşturmanıza olanak tanır. Bu, Svelte'in ön yüz gücü ile Laravel'in inanılmaz arka uç üretkenliğini ve yıldırım hızındaki Vite derlemesini bir arada kullanmanızı sağlar.

Svelte başlangıç kiti, Svelte 5, TypeScript, Tailwind ve shadcn-svelte bileşen kütüphanesini kullanır.

### Vue Başlangıç Kiti
Vue başlangıç kitimiz, Inertia kullanarak bir Vue ön yüzü ile Laravel uygulamaları oluşturmak için harika bir başlangıç noktası sağlar.

Inertia, klasik sunucu tarafı yönlendirme ve controller'ları kullanarak modern, tek sayfa Vue uygulamaları oluşturmanıza olanak tanır. Bu, Vue'un ön yüz gücü ile Laravel'in inanılmaz arka uç üretkenliğini ve yıldırım hızındaki Vite derlemesini bir arada kullanmanızı sağlar.

Vue başlangıç kiti, Vue Composition API, TypeScript, Tailwind ve shadcn-vue bileşen kütüphanesini kullanır.

### Livewire Başlangıç Kiti
Livewire başlangıç kitimiz, bir Laravel Livewire ön yüzü ile Laravel uygulamaları oluşturmak için mükemmel başlangıç noktasını sağlar.

Livewire, yalnızca PHP kullanarak dinamik, tepkisel (reactive) ön yüz kullanıcı arayüzleri (UI) oluşturmanın güçlü bir yoludur. Öncelikle Blade şablonları kullanan ve React, Vue veya Svelte gibi JavaScript odaklı SPA framework'lerine daha basit bir alternatif arayan ekipler için harika bir uyumdur.

Livewire başlangıç kiti, Livewire, Tailwind ve Flux UI bileşen kütüphanesini kullanır.

## Başlangıç Kiti Özelleştirme

### React Kiti Özelleştirme
React başlangıç kitimiz Inertia 2, React 19, Tailwind 4 ve shadcn/ui ile oluşturulmuştur. Tüm başlangıç kitlerimizde olduğu gibi, tam özelleştirmeye izin vermek için arka uç ve ön uç kodlarının tümü uygulamanız içinde bulunur.

Ön uç kodunun çoğu `resources/js` dizininde bulunur. Uygulamanızın görünümünü ve davranışını özelleştirmek için herhangi bir kodu değiştirmekte özgürsünüz:
```
resources/js/
├── components/    # Yeniden kullanılabilir React bileşenleri
├── hooks/         # React hook'ları
├── layouts/       # Uygulama düzenleri (layout)
├── lib/           # Yardımcı fonksiyonlar ve yapılandırma
├── pages/         # Sayfa bileşenleri
└── types/         # TypeScript tanımları
```

#### Ek shadcn Bileşenleri Yayınlama
Ek shadcn bileşenleri yayınlamak için önce yayınlamak istediğiniz bileşeni bulun. Ardından, `npx` kullanarak bileşeni yayınlayın:
```bash
npx shadcn@latest add switch
```
Bu örnekte, komut Switch bileşenini `resources/js/components/ui/switch.tsx` konumuna yayınlayacaktır. Bileşen yayınlandıktan sonra, onu sayfalarınızın herhangi birinde kullanabilirsiniz:
```jsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

#### Mevcut Düzenler (Layouts)
React başlangıç kiti, seçebileceğiniz iki farklı ana düzen içerir: bir "kenar çubuğu" (sidebar) düzeni ve bir "başlık" (header) düzeni. Kenar çubuğu düzeni varsayılandır, ancak uygulamanızın `resources/js/layouts/app-layout.tsx` dosyasının en üstünde içe aktarılan düzeni değiştirerek başlık düzenine geçebilirsiniz:
```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout';
import AppLayoutTemplate from '@/layouts/app/app-header-layout';
```

#### Kenar Çubuğu Varyantları
Kenar çubuğu düzeni üç farklı varyant içerir: varsayılan kenar çubuğu varyantı, "içe geçik" (inset) varyantı ve "yüzen" (floating) varyantı. `resources/js/components/app-sidebar.tsx` bileşenini değiştirerek en çok beğendiğiniz varyantı seçebilirsiniz:
```jsx
<Sidebar collapsible="icon" variant="sidebar">
<Sidebar collapsible="icon" variant="inset">
```

#### Kimlik Doğrulama Sayfası Düzen Varyantları
React başlangıç kitiyle birlikte gelen giriş sayfası ve kayıt sayfası gibi kimlik doğrulama sayfaları da üç farklı düzen varyantı sunar: "basit" (simple), "kart" (card) ve "bölünmüş" (split).

Kimlik doğrulama düzeninizi değiştirmek için uygulamanızın `resources/js/layouts/auth-layout.tsx` dosyasının en üstünde içe aktarılan düzeni değiştirin:
```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout';
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout';
```

### Svelte Kiti Özelleştirme
Svelte başlangıç kitimiz Inertia 2, Svelte 5, Tailwind ve shadcn-svelte ile oluşturulmuştur. Tüm başlangıç kitlerimizde olduğu gibi, tam özelleştirmeye izin vermek için arka uç ve ön uç kodlarının tümü uygulamanız içinde bulunur.

Ön uç kodunun çoğu `resources/js` dizininde bulunur. Uygulamanızın görünümünü ve davranışını özelleştirmek için herhangi bir kodu değiştirmekte özgürsünüz:
```
resources/js/
├── components/    # Yeniden kullanılabilir Svelte bileşenleri
├── layouts/       # Uygulama düzenleri (layout)
├── lib/           # Yardımcı fonksiyonlar, yapılandırma ve Svelte rune modülleri
├── pages/         # Sayfa bileşenleri
└── types/         # TypeScript tanımları
```

#### Ek shadcn-svelte Bileşenleri Yayınlama
Ek shadcn-svelte bileşenleri yayınlamak için önce yayınlamak istediğiniz bileşeni bulun. Ardından, `npx` kullanarak bileşeni yayınlayın:
```bash
npx shadcn-svelte@latest add switch
```
Bu örnekte, komut Switch bileşenini `resources/js/components/ui/switch/switch.svelte` konumuna yayınlayacaktır. Bileşen yayınlandıktan sonra, onu sayfalarınızın herhangi birinde kullanabilirsiniz:
```svelte
<script lang="ts">
    import { Switch } from '@/components/ui/switch'
</script>

<div>
    <Switch />
</div>
```

#### Mevcut Düzenler (Layouts)
Svelte başlangıç kiti, seçebileceğiniz iki farklı ana düzen içerir: bir "kenar çubuğu" (sidebar) düzeni ve bir "başlık" (header) düzeni. Kenar çubuğu düzeni varsayılandır, ancak uygulamanızın `resources/js/layouts/AppLayout.svelte` dosyasının en üstünde içe aktarılan düzeni değiştirerek başlık düzenine geçebilirsiniz:
```js
import AppLayout from '@/layouts/app/AppSidebarLayout.svelte';
import AppLayout from '@/layouts/app/AppHeaderLayout.svelte';
```

#### Kenar Çubuğu Varyantları
Kenar çubuğu düzeni üç farklı varyant içerir: varsayılan kenar çubuğu varyantı, "içe geçik" (inset) varyantı ve "yüzen" (floating) varyantı. `resources/js/components/AppSidebar.svelte` bileşenini değiştirerek en çok beğendiğiniz varyantı seçebilirsiniz:
```svelte
<Sidebar collapsible="icon" variant="sidebar">
<Sidebar collapsible="icon" variant="inset">
```

#### Kimlik Doğrulama Sayfası Düzen Varyantları
Svelte başlangıç kitiyle birlikte gelen giriş sayfası ve kayıt sayfası gibi kimlik doğrulama sayfaları da üç farklı düzen varyantı sunar: "basit" (simple), "kart" (card) ve "bölünmüş" (split).

Kimlik doğrulama düzeninizi değiştirmek için uygulamanızın `resources/js/layouts/AuthLayout.svelte` dosyasının en üstünde içe aktarılan düzeni değiştirin:
```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.svelte';
import AuthLayout from '@/layouts/auth/AuthSplitLayout.svelte';
```

### Vue Kiti Özelleştirme
Vue başlangıç kitimiz Inertia 2, Vue 3 Composition API, Tailwind ve shadcn-vue ile oluşturulmuştur. Tüm başlangıç kitlerimizde olduğu gibi, tam özelleştirmeye izin vermek için arka uç ve ön uç kodlarının tümü uygulamanız içinde bulunur.

Ön uç kodunun çoğu `resources/js` dizininde bulunur. Uygulamanızın görünümünü ve davranışını özelleştirmek için herhangi bir kodu değiştirmekte özgürsünüz:
```
resources/js/
├── components/    # Yeniden kullanılabilir Vue bileşenleri
├── composables/   # Vue composable'ları / hook'ları
├── layouts/       # Uygulama düzenleri (layout)
├── lib/           # Yardımcı fonksiyonlar ve yapılandırma
├── pages/         # Sayfa bileşenleri
└── types/         # TypeScript tanımları
```

#### Ek shadcn-vue Bileşenleri Yayınlama
Ek shadcn-vue bileşenleri yayınlamak için önce yayınlamak istediğiniz bileşeni bulun. Ardından, `npx` kullanarak bileşeni yayınlayın:
```bash
npx shadcn-vue@latest add switch
```
Bu örnekte, komut Switch bileşenini `resources/js/components/ui/Switch.vue` konumuna yayınlayacaktır. Bileşen yayınlandıktan sonra, onu sayfalarınızın herhangi birinde kullanabilirsiniz:
```vue
<script setup lang="ts">
import { Switch } from '@/components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

#### Mevcut Düzenler (Layouts)
Vue başlangıç kiti, seçebileceğiniz iki farklı ana düzen içerir: bir "kenar çubuğu" (sidebar) düzeni ve bir "başlık" (header) düzeni. Kenar çubuğu düzeni varsayılandır, ancak uygulamanızın `resources/js/layouts/AppLayout.vue` dosyasının en üstünde içe aktarılan düzeni değiştirerek başlık düzenine geçebilirsiniz:
```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue';
import AppLayout from '@/layouts/app/AppHeaderLayout.vue';
```

#### Kenar Çubuğu Varyantları
Kenar çubuğu düzeni üç farklı varyant içerir: varsayılan kenar çubuğu varyantı, "içe geçik" (inset) varyantı ve "yüzen" (floating) varyantı. `resources/js/components/AppSidebar.vue` bileşenini değiştirerek en çok beğendiğiniz varyantı seçebilirsiniz:
```vue
<Sidebar collapsible="icon" variant="sidebar">
<Sidebar collapsible="icon" variant="inset">
```

#### Kimlik Doğrulama Sayfası Düzen Varyantları
Vue başlangıç kitiyle birlikte gelen giriş sayfası ve kayıt sayfası gibi kimlik doğrulama sayfaları da üç farklı düzen varyantı sunar: "basit" (simple), "kart" (card) ve "bölünmüş" (split).

Kimlik doğrulama düzeninizi değiştirmek için uygulamanızın `resources/js/layouts/AuthLayout.vue` dosyasının en üstünde içe aktarılan düzeni değiştirin:
```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue';
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue';
```

### Livewire Kiti Özelleştirme
Livewire başlangıç kitimiz Livewire 4, Tailwind ve Flux UI ile oluşturulmuştur. Tüm başlangıç kitlerimizde olduğu gibi, tam özelleştirmeye izin vermek için arka uç ve ön uç kodlarının tümü uygulamanız içinde bulunur.

Ön uç kodunun çoğu `resources/views` dizininde bulunur. Uygulamanızın görünümünü ve davranışını özelleştirmek için herhangi bir kodu değiştirmekte özgürsünüz:
```
resources/views
├── components            # Yeniden kullanılabilir bileşenler
├── flux                  # Özelleştirilmiş Flux bileşenleri
├── layouts               # Uygulama düzenleri (layout)
├── pages                 # Livewire sayfaları
├── partials              # Yeniden kullanılabilir Blade parçaları (partials)
├── dashboard.blade.php   # Kimliği doğrulanmış kullanıcı paneli (dashboard)
└── welcome.blade.php     # Misafir kullanıcı karşılama sayfası
```

#### Mevcut Düzenler (Layouts)
Livewire başlangıç kiti, seçebileceğiniz iki farklı ana düzen içerir: bir "kenar çubuğu" (sidebar) düzeni ve bir "başlık" (header) düzeni. Kenar çubuğu düzeni varsayılandır, ancak uygulamanızın `resources/views/layouts/app.blade.php` dosyasında kullanılan düzeni değiştirerek başlık düzenine geçebilirsiniz. Ayrıca, ana Flux bileşenine `container` özniteliğini (attribute) eklemelisiniz:
```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

#### Kimlik Doğrulama Sayfası Düzen Varyantları
Livewire başlangıç kitiyle birlikte gelen giriş sayfası ve kayıt sayfası gibi kimlik doğrulama sayfaları da üç farklı düzen varyantı sunar: "basit" (simple), "kart" (card) ve "bölünmüş" (split).

Kimlik doğrulama düzeninizi değiştirmek için uygulamanızın `resources/views/layouts/auth.blade.php` dosyasında kullanılan düzeni değiştirin:
```blade
<x-layouts::auth.split>
    {{ $slot }}
</x-layouts::auth.split>
```

## Kimlik Doğrulama (Authentication)

Tüm başlangıç kitleri, kimlik doğrulamayı (authentication) yönetmek için Laravel Fortify'i kullanır. Fortify; giriş, kayıt, parola sıfırlama, e-posta doğrulama ve daha fazlası için rotalar (routes), controller'lar ve mantık sağlar.

Fortify, uygulamanızın `config/fortify.php` yapılandırma dosyasında etkinleştirilen özelliklere (features) bağlı olarak aşağıdaki kimlik doğrulama rotalarını otomatik olarak kaydeder:

| Rota | Metod | Açıklama |
| :--- | :--- | :--- |
| `/login` | GET | Giriş formunu görüntüle |
| `/login` | POST | Kullanıcının kimliğini doğrula |
| `/logout` | POST | Kullanıcının oturumunu kapat |
| `/register` | GET | Kayıt formunu görüntüle |
| `/register` | POST | Yeni kullanıcı oluştur |
| `/forgot-password` | GET | Parola sıfırlama istek formunu görüntüle |
| `/forgot-password` | POST | Parola sıfırlama bağlantısını gönder |
| `/reset-password/{token}` | GET | Parola sıfırlama formunu görüntüle |
| `/reset-password` | POST | Parolayı güncelle |
| `/email/verify` | GET | E-posta doğrulama bildirimini görüntüle |
| `/email/verify/{id}/{hash}` | GET | E-posta adresini doğrula |
| `/email/verification-notification` | POST | Doğrulama e-postasını yeniden gönder |
| `/user/confirm-password` | GET | Parola onaylama formunu görüntüle |
| `/user/confirm-password` | POST | Parolayı onayla |
| `/two-factor-challenge` | GET | 2FA (İki Faktörlü Doğrulama) sınav formunu görüntüle |