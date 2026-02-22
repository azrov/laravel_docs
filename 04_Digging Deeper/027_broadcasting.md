# Laravel 12 Dokümantasyonu: Yayıncılık (Broadcasting)

## Giriş

Birçok modern web uygulamasında, gerçek zamanlı (realtime), canlı güncellenen kullanıcı arayüzleri uygulamak için WebSockets kullanılır. Sunucuda bazı veriler güncellendiğinde, istemci (client) tarafından işlenmek üzere tipik olarak bir WebSocket bağlantısı üzerinden bir mesaj gönderilir. WebSockets, kullanıcı arayüzünüzde yansıtılması gereken veri değişiklikleri için uygulamanızın sunucusunu sürekli olarak yoklamaya (polling) kıyasla daha verimli bir alternatif sunar.

Örneğin, uygulamanızın bir kullanıcının verilerini CSV dosyasına aktarabildiğini ve e-posta ile gönderebildiğini hayal edin. Ancak, bu CSV dosyasını oluşturmak birkaç dakika sürüyor, bu nedenle CSV'yi sıraya konmuş bir iş (queued job) içinde oluşturup göndermeyi seçiyorsunuz. CSV oluşturulup kullanıcıya e-posta ile gönderildiğinde, olay yayıncılığını (event broadcasting) kullanarak uygulamamızın JavaScript'i tarafından alınacak bir `App\Events\UserDataExported` olayı gönderebiliriz. Olay alındıktan sonra, kullanıcıya CSV'lerinin e-posta ile gönderildiğini, sayfayı yenilemelerine gerek kalmadan bir mesajla gösterebiliriz.

Bu tür özellikleri oluşturmanıza yardımcı olmak için Laravel, sunucu tarafındaki Laravel olaylarınızı bir WebSocket bağlantısı üzerinden "yayınlamayı" (broadcast) kolaylaştırır. Laravel olaylarınızı yayınlamak, aynı olay adlarını ve verilerini sunucu tarafındaki Laravel uygulamanız ile istemci tarafındaki JavaScript uygulamanız arasında paylaşmanıza olanak tanır.

Yayıncılığın arkasındaki temel kavramlar basittir: istemciler ön uçta (frontend) adlandırılmış kanallara (channels) bağlanırken, Laravel uygulamanız arka uçta (backend) bu kanallara olaylar yayınlar. Bu olaylar, ön uca sunmak istediğiniz herhangi bir ek veriyi içerebilir.

#### Desteklenen Sürücüler (Supported Drivers)
Varsayılan olarak Laravel, aralarından seçim yapabileceğiniz üç sunucu tarafı yayıncılık sürücüsü (server-side broadcasting drivers) içerir: Laravel Reverb, Pusher Channels ve Ably.

Olay yayıncılığına dalmadan önce, Laravel'in olaylar (events) ve dinleyiciler (listeners) hakkındaki dokümantasyonunu okuduğunuzdan emin olun.

## Hızlı Başlangıç (Quickstart)

Varsayılan olarak, yeni Laravel uygulamalarında yayıncılık (broadcasting) etkin değildir. `install:broadcasting` Artisan komutunu kullanarak yayıncılığı etkinleştirebilirsiniz:
```bash
php artisan install:broadcasting
```
`install:broadcasting` komutu size hangi olay yayıncılık servisini kullanmak istediğinizi soracaktır. Ayrıca, `config/broadcasting.php` yapılandırma dosyasını ve uygulamanızın yayın yetkilendirme rotalarını (broadcast authorization routes) ve geri çağrılarını (callbacks) kaydedebileceğiniz `routes/channels.php` dosyasını oluşturacaktır.

Laravel kutudan çıktığı gibi birkaç yayın sürücüsünü (broadcast drivers) destekler: Laravel Reverb, Pusher Channels, Ably ve yerel geliştirme ve hata ayıklama için bir `log` sürücüsü. Ayrıca, test sırasında yayıncılığı devre dışı bırakmanıza izin veren bir `null` sürücüsü de dahildir. Bu sürücülerin her biri için `config/broadcasting.php` yapılandırma dosyasında bir yapılandırma örneği bulunur.

Uygulamanızın tüm olay yayıncılık yapılandırması `config/broadcasting.php` yapılandırma dosyasında saklanır. Bu dosya uygulamanızda yoksa endişelenmeyin; `install:broadcasting` Artisan komutunu çalıştırdığınızda oluşturulacaktır.

#### Sonraki Adımlar (Next Steps)
Olay yayıncılığını etkinleştirdikten sonra, yayın olaylarını (broadcast events) tanımlama ve olayları dinleme hakkında daha fazla bilgi edinmeye hazırsınız. Laravel'in React veya Vue başlangıç kitlerini (starter kits) kullanıyorsanız, Echo'nun `useEcho` hook'unu kullanarak olayları dinleyebilirsiniz.

Herhangi bir olayı yayınlamadan önce, bir kuyruk çalışanını (queue worker) yapılandırmalı ve çalıştırmalısınız. Tüm olay yayıncılığı, yayınlanan olaylar nedeniyle uygulamanızın yanıt süresinin ciddi şekilde etkilenmemesi için sıraya konmuş işler (queued jobs) aracılığıyla yapılır.

## Sunucu Tarafı Kurulumu (Server Side Installation)

Laravel'in olay yayıncılığını kullanmaya başlamak için Laravel uygulaması içinde biraz yapılandırma yapmamız ve birkaç paket kurmamız gerekiyor.

Olay yayıncılığı, Laravel olaylarınızı yayınlayan sunucu tarafı bir yayın sürücüsü (server-side broadcasting driver) tarafından gerçekleştirilir, böylece Laravel Echo (bir JavaScript kütüphanesi) bunları tarayıcı istemcisi içinde alabilir. Endişelenmeyin - kurulum sürecinin her bölümünde adım adım ilerleyeceğiz.

### Reverb Kurulumu
Olay yayıncınız (event broadcaster) olarak Reverb'ü kullanırken Laravel'in yayıncılık özellikleri için desteği hızlıca etkinleştirmek üzere `install:broadcasting` Artisan komutunu `--reverb` seçeneğiyle çağırın. Bu Artisan komutu, Reverb'ün gerekli Composer ve NPM paketlerini kuracak ve uygulamanızın `.env` dosyasını uygun değişkenlerle güncelleyecektir:
```bash
php artisan install:broadcasting --reverb
```

#### Manuel Kurulum (Manual Installation)
`install:broadcasting` komutunu çalıştırırken, Laravel Reverb'ü kurmanız istenecektir. Elbette, Reverb'ü Composer paket yöneticisini kullanarak manuel olarak da kurabilirsiniz:
```bash
composer require laravel/reverb
```
Paket kurulduktan sonra, yapılandırmayı yayınlamak, Reverb'ün gerekli ortam değişkenlerini eklemek ve uygulamanızda olay yayıncılığını etkinleştirmek için Reverb'ün kurulum komutunu çalıştırabilirsiniz:
```bash
php artisan reverb:install
```
Ayrıntılı Reverb kurulum ve kullanım talimatlarını Reverb dokümantasyonunda bulabilirsiniz.

### Pusher Channels Kurulumu
Olay yayıncınız olarak Pusher'ı kullanırken Laravel'in yayıncılık özellikleri için desteği hızlıca etkinleştirmek üzere `install:broadcasting` Artisan komutunu `--pusher` seçeneğiyle çağırın. Bu Artisan komutu size Pusher kimlik bilgilerinizi (credentials) soracak, Pusher PHP ve JavaScript SDK'larını kuracak ve uygulamanızın `.env` dosyasını uygun değişkenlerle güncelleyecektir:
```bash
php artisan install:broadcasting --pusher
```

#### Manuel Kurulum (Manual Installation)
Pusher desteğini manuel olarak kurmak için Pusher Channels PHP SDK'sını Composer paket yöneticisini kullanarak kurmalısınız:
```bash
composer require pusher/pusher-php-server
```
Ardından, Pusher Channels kimlik bilgilerinizi `config/broadcasting.php` yapılandırma dosyasında yapılandırmalısınız. Bu dosyada zaten bir örnek Pusher Channels yapılandırması bulunur, bu da anahtarınızı (key), sırrınızı (secret) ve uygulama ID'nizi hızlıca belirtmenize olanak tanır. Tipik olarak, Pusher Channels kimlik bilgilerinizi uygulamanızın `.env` dosyasında yapılandırmalısınız:
```
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"
```
`config/broadcasting.php` dosyasındaki `pusher` yapılandırması ayrıca `cluster` gibi Channels tarafından desteklenen ek seçenekleri belirtmenize de olanak tanır.

Ardından, uygulamanızın `.env` dosyasında `BROADCAST_CONNECTION` ortam değişkenini `pusher` olarak ayarlayın:
```
BROADCAST_CONNECTION=pusher
```
Son olarak, istemci tarafında yayın olaylarını alacak olan Laravel Echo'yu kurmaya ve yapılandırmaya hazırsınız.

### Ably Kurulumu
Aşağıdaki dokümantasyon, Ably'nin "Pusher uyumluluk" (Pusher compatibility) modunda nasıl kullanılacağını tartışır. Ancak, Ably ekibi Ably tarafından sunulan benzersiz yeteneklerden yararlanabilen bir yayıncı (broadcaster) ve Echo istemcisi önerir ve sürdürür. Ably tarafından sürdürülen sürücüleri kullanma hakkında daha fazla bilgi için lütfen Ably'nin Laravel yayıncı dokümantasyonuna bakın.

Olay yayıncınız olarak Ably'yi kullanırken Laravel'in yayıncılık özellikleri için desteği hızlıca etkinleştirmek üzere `install:broadcasting` Artisan komutunu `--ably` seçeneğiyle çağırın. Bu Artisan komutu size Ably kimlik bilgilerinizi soracak, Ably PHP ve JavaScript SDK'larını kuracak ve uygulamanızın `.env` dosyasını uygun değişkenlerle güncelleyecektir:
```bash
php artisan install:broadcasting --ably
```
Devam etmeden önce, Ably uygulama ayarlarınızda Pusher protokol desteğini (Pusher protocol support) etkinleştirmelisiniz. Bu özelliği, Ably uygulamanızın ayarlar panelindeki "Protocol Adapter Settings" bölümü içinde etkinleştirebilirsiniz.

#### Manuel Kurulum (Manual Installation)
Ably desteğini manuel olarak kurmak için Ably PHP SDK'sını Composer paket yöneticisini kullanarak kurmalısınız:
```bash
composer require ably/ably-php
```
Ardından, Ably kimlik bilgilerinizi `config/broadcasting.php` yapılandırma dosyasında yapılandırmalısınız. Bu dosyada zaten bir örnek Ably yapılandırması bulunur, bu da anahtarınızı (key) hızlıca belirtmenize olanak tanır. Tipik olarak bu değer `ABLY_KEY` ortam değişkeni aracılığıyla ayarlanmalıdır:
```
ABLY_KEY=your-ably-key
```
Ardından, uygulamanızın `.env` dosyasında `BROADCAST_CONNECTION` ortam değişkenini `ably` olarak ayarlayın:
```
BROADCAST_CONNECTION=ably
```
Son olarak, istemci tarafında yayın olaylarını alacak olan Laravel Echo'yu kurmaya ve yapılandırmaya hazırsınız.

## İstemci Tarafı Kurulumu (Client Side Installation)

### Reverb için İstemci Kurulumu
Laravel Echo, sunucu tarafı yayın sürücünüz tarafından yayınlanan olayları dinlemek ve kanallara abone olmayı (subscribe) zahmetsiz hale getiren bir JavaScript kütüphanesidir.

`install:broadcasting` Artisan komutu aracılığıyla Laravel Reverb'ü kurarken, Reverb ve Echo'nun iskelesi (scaffolding) ve yapılandırması uygulamanıza otomatik olarak enjekte edilecektir. Ancak, Laravel Echo'yu manuel olarak yapılandırmak isterseniz, bunu aşağıdaki talimatları izleyerek yapabilirsiniz.

#### Manuel Kurulum (Manual Installation)
Uygulamanızın ön ucu için Laravel Echo'yu manuel olarak yapılandırmak üzere, önce `pusher-js` paketini kurun, çünkü Reverb WebSocket abonelikleri, kanallar ve mesajlar için Pusher protokolünü kullanır:
```bash
npm install --save-dev laravel-echo pusher-js
```
Echo kurulduktan sonra, uygulamanızın JavaScript'inde yeni bir Echo örneği (instance) oluşturmaya hazırsınız. Bunu yapmak için harika bir yer, Laravel framework'ü ile birlikte gelen `resources/js/bootstrap.js` dosyasının altıdır:
```javascript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```
Ardından, uygulamanızın varlıklarını (assets) derlemelisiniz:
```bash
npm run build
```
Laravel Echo `reverb` yayıncısı, `laravel-echo` v1.16.0+ gerektirir.

### Pusher Channels için İstemci Kurulumu
`install:broadcasting --pusher` Artisan komutu aracılığıyla yayıncılık desteğini kurarken, Pusher ve Echo'nun iskelesi ve yapılandırması uygulamanıza otomatik olarak enjekte edilecektir. Ancak, Laravel Echo'yu manuel olarak yapılandırmak isterseniz, bunu aşağıdaki talimatları izleyerek yapabilirsiniz.

#### Manuel Kurulum (Manual Installation)
Uygulamanızın ön ucu için Laravel Echo'yu manuel olarak yapılandırmak üzere, önce WebSocket abonelikleri, kanallar ve mesajlar için Pusher protokolünü kullanan `laravel-echo` ve `pusher-js` paketlerini kurun:
```bash
npm install --save-dev laravel-echo pusher-js
```
Echo kurulduktan sonra, uygulamanızın `resources/js/bootstrap.js` dosyasında yeni bir Echo örneği oluşturmaya hazırsınız:
```javascript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```
Ardından, uygulamanızın `.env` dosyasında Pusher ortam değişkenleri için uygun değerleri tanımlamalısınız. Bu değişkenler `.env` dosyanızda zaten yoksa, bunları eklemelisiniz:
```
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"

VITE_APP_NAME="${APP_NAME}"
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```
Echo yapılandırmasını uygulamanızın ihtiyaçlarına göre ayarladıktan sonra, uygulamanızın varlıklarını derleyebilirsiniz:
```bash
npm run build
```
Uygulamanızın JavaScript varlıklarını derleme hakkında daha fazla bilgi edinmek için lütfen Vite ile ilgili dokümantasyona bakın.

#### Mevcut Bir İstemci Örneğini Kullanma (Using an Existing Client Instance)
Echo'nun kullanmasını istediğiniz önceden yapılandırılmış bir Pusher Channels istemci örneğine (client instance) zaten sahipseniz, bunu `client` yapılandırma seçeneği aracılığıyla Echo'ya iletebilirsiniz:
```javascript
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```